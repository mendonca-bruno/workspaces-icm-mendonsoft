# Stage 03: Busca de Contexto (cases primeiro, KB hierárquica depois)

Encontrar o conjunto mínimo e suficiente de cases + dossiês + playbooks que dá contexto para resolver o ticket. **Cases têm prioridade absoluta** sobre dossiês de KB.

## Inputs

| Fonte | Arquivo/Localização | Escopo | Por quê |
|-------|---------------------|--------|---------|
| Ticket clarificado | `../../tickets/<id>/02-clarified.md` | Frontmatter + corpo | Critério de busca |
| Index de cases | `../../cases/INDEX.md` | Tabela completa | Buscar cases similares (prioridade 1) |
| Index de KB | `../../kb/INDEX.md` | Tabela completa | Cardápio de dossiês (prioridade 2) |
| Topic schema | `../../kb/_taxonomies/topic-schema-by-area.md` | Seção da área | Query expansion |
| Roteadores de tecnologias suspeitas | `../../kb/<tech>/CONTEXT.md` | Apenas das techs em `systems_involved` | Descobrir sistemas/dossiês |
| Roteadores de sistemas suspeitos | `../../kb/<tech>/<sistema>/CONTEXT.md` | Apenas dos sistemas relevantes | Listar dossiês candidatos |
| Playbooks | `../../kb/_playbooks/*.md` | Apenas com sintomas matching | Padrões consolidados |

## Processo

### Camada 1 — Cases primeiro (a maioria dos tickets resolve aqui)

1. Filtrar `cases/INDEX.md` por `Sistemas` ∩ `02-clarified.systems_involved` E `Sintomas` ∩ `02-clarified.symptoms_normalized`.
2. Calcular score de match por caso candidato:
   - +2 por sintoma em comum
   - +1 por sistema em comum
   - +1 se `area` em comum
3. **Hit forte**: ≥1 caso com score ≥ 5 (ex.: 2 sintomas + 1 sistema). Carregar caso(s) full + dossiês listados em `dossiers_cited:`.
4. Se hit forte → emitir `03-search-result.md` com `case_match: <id>` e seguir para Stage 04 (não precisa de mais busca).

### Camada 2 — Query expansion + roteamento hierárquico

5. Sem hit forte: aplicar **query expansion** via `topic-schema-by-area.md` da área:
   - Expandir entidades (ex.: "vale" → [VR, VA, VT, BENEFICIO]).
   - Expandir sintomas com aliases.
   - Mapear códigos de erro mencionados → entidades canônicas.
6. **Roteamento hierárquico**:
   - Para cada `tech` em `systems_involved`: ler `kb/<tech>/CONTEXT.md`.
   - Para cada `sistema`: ler `kb/<tech>/<sistema>/CONTEXT.md`.
   - Selecionar 1-3 dossiês candidatos por sistema (não mais).
7. Carregar dossiês candidatos full. Ler frontmatters para validar que `entities` e `symptoms` declarados batem com a expansão.

### Camada 3 — Cross-references e playbooks

8. Para cada dossiê carregado, expandir set via `related:` se contexto incompleto (ex.: dossiê de schema referencia dossiê de fluxo de integração).
9. Verificar se algum sintoma matching tem playbook em `kb/_playbooks/`. Se sim, carregar.

### Emissão

10. Compor `03-search-result.md` ranqueado:
    - Se Camada 1 deu hit: `case_match: <id>` + cases citados + dossiês de `dossiers_cited:`.
    - Se Camada 2/3: lista ranqueada de dossiês com snippet de "por que carregado" + playbooks.
    - Cada item com path absoluto e seção do dossiê mais relevante.

## Verify

| Verificação | Comando | Condição de Aprovação |
|-------------|---------|----------------------|
| Cita ≥1 fonte | grep em `03-search-result.md` | Pelo menos 1 path para `cases/` ou `kb/` |
| Sistemas alinhados | Cada dossiê citado tem `systems:` ∩ `02-clarified.systems_involved` ≠ ∅ | Inspecionar frontmatters |
| Sem dossiês fora do escopo | Nenhum dossiê citado de tech NÃO listada em `systems_involved` | grep |
| Cases primeiro foi tentado | `03-search-result.md` tem seção "## Cases consultados" com a tabela de scores | Inspecionar |

## Audit

| Verificação | Condição de Aprovação |
|-------------|----------------------|
| Carregamento seletivo | Total de dossiês carregados ≤ 5 (sinal de roteamento funcionando) |
| Cases priorizados | Quando `case_match` existe, `03-search-result.md` cita o caso ANTES de qualquer dossiê |
| Cross-references seguidas | Se 1+ dossiê carregado tem `related:` apontando para outro relevante, esse outro está carregado |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Resultado da busca | `../../tickets/<id>/03-search-result.md` | Markdown com seções: `## Cases consultados`, `## Hit forte` ou `## Dossiês candidatos`, `## Playbooks`, `## Caminho de raciocínio` |
