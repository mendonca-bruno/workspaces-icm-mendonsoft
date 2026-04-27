# Stage 06: Manutenção da KB (offline, semanal)

Auditoria periódica da saúde da KB: detectar drift de hashes (sources mudaram desde última destilação), dossiês antigos, clusters de cases não promovidos, contradições pendentes, dossiês órfãos. Regenera índices.

**Quando rodar**: semanalmente (ex.: sexta-feira), ou sob demanda quando algo parece desatualizado.

## Inputs

| Fonte | Arquivo/Localização | Escopo | Por quê |
|-------|---------------------|--------|---------|
| Sources canônicos | `../../kb/_sources/**/*` | Listagem + hash atual | Detectar drift |
| Dossiês | `../../kb/<tech>/<sistema>/*.md` | Apenas frontmatters | Comparar `source_sha256` e `last_distilled` |
| Index de cases | `../../cases/INDEX.md` | Tabela completa | Detectar clusters não promovidos |
| Pending-updates | `../../kb/_pending-updates.md` | Arquivo completo | Flags abertas pelo Stage 05 |
| PROGRESS | `../../PROGRESS.md` | Métricas de uso (se hit-counter está implementado) | Detectar dossiês com 0 hits |

## Processo

### Auditoria

1. **Drift de hashes**:
   - Para cada dossiê com `source:` no frontmatter, ler o source canônico em `kb/_sources/...` e calcular SHA-256 atual.
   - Se hash atual ≠ `source_sha256` do frontmatter: marcar como **drift**.
2. **Dossiês desatualizados**:
   - Filtrar dossiês com `last_distilled` > 90 dias atrás.
   - Listar separadamente: drift + antigos vs. apenas antigos.
3. **Clusters de cases para promoção**:
   - Agrupar `cases/INDEX.md` por `(area, systems_involved, symptoms)`.
   - Filtrar grupos com ≥3 cases criados nos últimos 90 dias E nenhum com `promoted_to:`.
4. **Contradições pendentes**:
   - Ler `kb/_pending-updates.md`. Listar todas as entradas abertas.
5. **Dossiês órfãos**:
   - Filtrar dossiês com 0 hits no PROGRESS (se hit-counter implementado) E nenhum case os cita em `dossiers_cited:` nos últimos 180 dias.
   - Candidatos a `kb/_archive/`.

### Regeneração de índices

6. **Regenerar `kb/INDEX.md`**: ler frontmatter de TODO `kb/<tech>/<sistema>/*.md` e renderizar a tabela agregada.
7. **Regenerar `cases/INDEX.md`**: ler frontmatter de TODO `cases/INC*.md` e renderizar a tabela agregada.

### Compor relatório

8. Emitir `kb-health-report.md` na raiz do workspace com seções:
   - `## Drift detectado` (lista dossiê + source + hash old/new)
   - `## Dossiês desatualizados` (lista dossiê + last_distilled)
   - `## Promoções candidatas` (lista cluster + cases que o compõem + sugestão de playbook name)
   - `## Contradições pendentes` (do `_pending-updates.md`)
   - `## Dossiês órfãos` (candidatos a arquivar)

### Checkpoint e ação

9. **[Checkpoint]** — Bruno revisa o report. Decide:
   - Quais dossiês re-destilar (re-rodar Stage 01 com os sources que driftaram).
   - Quais clusters promover (re-rodar Stage 05 modo `promote`).
   - Quais órfãos arquivar.
   - Quais contradições resolver agora vs. adiar.
10. Após decisões, executar (ou agendar) as ações. Bruno marca `kb/_pending-updates.md` removendo entradas resolvidas.

## Checkpoints

| Após Passo | Agente Apresenta | Humano Decide |
|------------|-------------------|---------------|
| 8 | `kb-health-report.md` completo | Quais ações tomar (re-destilar, promover, arquivar, resolver contradição) |

## Verify

| Verificação | Comando | Condição de Aprovação |
|-------------|---------|----------------------|
| Hashes recalculados | Para amostra aleatória de 5 dossiês, hash do report bate com `sha256sum` do source | Manual |
| INDEX.md atualizado | Conta de linhas == número de dossiês existentes | wc -l |
| cases/INDEX.md atualizado | Conta de linhas == número de cases existentes | wc -l |

## Audit

| Verificação | Condição de Aprovação |
|-------------|----------------------|
| Sem dossiê esquecido | Todo arquivo `kb/<tech>/<sistema>/*.md` (exceto CONTEXT.md) aparece em `INDEX.md` |
| Sem case esquecido | Todo arquivo `cases/INC*.md` aparece em `cases/INDEX.md` |
| Pending-updates não acumula | Entradas em `_pending-updates.md` não devem ter > 30 dias sem decisão |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Relatório de saúde | `../../kb-health-report.md` | Markdown com seções listadas no passo 8 |
| INDEX agregado de KB | `../../kb/INDEX.md` | Tabela markdown regenerada |
| INDEX agregado de cases | `../../cases/INDEX.md` | Tabela markdown regenerada |
