# Stage 02: Clarificação (U-shape com Human Edit Gate)

Normalizar o ticket bruto em uma representação acionável (sistema(s) identificado(s), sintoma(s) canônico(s), entidades suspeitas, janela temporal). Se o ticket é vago, **PARAR** o pipeline e gerar perguntas estruturadas para o solicitante.

Princípio: pipeline NÃO avança sem clarificação suficiente. Tickets vagos viram clarificação, não improvisação (paper ICM §heavy-edit stage 1).

## Inputs

| Fonte | Arquivo/Localização | Escopo | Por quê |
|-------|---------------------|--------|---------|
| Ticket bruto | `../../tickets/<id>/input.md` | Arquivo completo | O ticket a clarificar |
| Taxonomia de sintomas | `../../kb/_taxonomies/symptoms.md` | Lista canônica + aliases | Normalizar a frase do solicitante |
| Mapa área × sistemas | `../../kb/_taxonomies/area-mapping.md` | Tabela completa | Identificar sistemas suspeitos |
| Topic schema | `../../kb/_taxonomies/topic-schema-by-area.md` | Seção da área provável | Entidades, sinônimos, códigos |
| Index de cases | `../../cases/INDEX.md` | Tabela completa | Verificar se sintoma já é conhecido |
| Rubrica de clarificação | `../../reference/clarification-rubric.md` | Arquivo completo | Critérios do score |

## Processo

1. Ler `tickets/<id>/input.md`.
2. Extrair **frases-sintoma brutas** (o que o solicitante disse literalmente).
3. **Normalizar contra `symptoms.md`**: para cada frase, encontrar o `id` canônico mais próximo via aliases. Se nenhum match: marcar como `symptom_unknown` (e flagar para adicionar à taxonomia depois).
4. **Identificar sistemas suspeitos**: cruzar entidades mencionadas, aliases de sistemas, área implícita pelo solicitante (RH disse "folha"? "ponto"?).
5. **Calcular score de clarificação** (4 dimensões — ver `clarification-rubric.md`):
   - **Sistema identificado** (0/1): pelo menos 1 sistema do `area-mapping.md` é nomeado ou implícito.
   - **Sintoma normalizado** (0/1): pelo menos 1 sintoma da taxonomia foi matched (não `symptom_unknown`).
   - **Janela temporal** (0/1): há indicação de quando o problema começou ou em qual ciclo (ex.: "fechamento de março", "desde ontem").
   - **Identificador afetado** (0/1): há matrícula, CPF, ID de processo, ou abrangência clara (ex.: "todos os funcionários da CCT X").
6. **Decisão**:
   - Score ≥ threshold (default: 3 de 4) → produzir `02-clarified.md` e prosseguir para Stage 03.
   - Score < threshold → **gerar `02-clarification.md`** com perguntas estruturadas (uma por dimensão faltante) e **PARAR**.
7. **[Checkpoint — Bloqueante]** Quando paramos por score baixo:
   - Bruno revisa `02-clarification.md` (pode editar perguntas), copia para o solicitante.
   - Solicitante responde; Bruno cola a resposta no MESMO arquivo `02-clarification.md` em uma seção `## Resposta`.
   - Bruno re-dispara `clarificar <ticket-id>` → loop ao passo 2 com input enriquecido (`input.md` + resposta).
8. Quando score OK, **gerar `02-clarified.md`** com:
   - `systems_involved: [...]` (lista canônica do `area-mapping.md`)
   - `symptoms_normalized: [...]` (lista canônica do `symptoms.md`)
   - `entities_suspected: [...]` (do `topic-schema-by-area.md`)
   - `area: <folha|ponto|beneficios|admissao|demissao>`
   - `severity: <P1|P2|P3|P4>` (Bruno marca)
   - `clarification_score: <N>/4`
   - `original_quote:` resumo de 2-3 linhas do ticket original

## Checkpoints

| Após Passo | Agente Apresenta | Humano Decide |
|------------|-------------------|---------------|
| 6 | Score calculado + dimensões faltantes; ou clarificação completa | Aprovar avanço, ou aprovar perguntas geradas |
| 7 | Perguntas em `02-clarification.md` | Editar/refinar antes de mandar; aguardar resposta antes de re-disparar |

## Verify

| Verificação | Comando | Condição de Aprovação |
|-------------|---------|----------------------|
| Score ≥ threshold | Inspecionar `02-clarified.md` frontmatter | `clarification_score:` ≥ 3 (ou threshold configurado) |
| Sintomas canônicos | Cada item em `symptoms_normalized:` deve existir como `id` em `symptoms.md` | grep |
| Sistemas canônicos | Cada item em `systems_involved:` deve existir em `area-mapping.md` | grep |

## Audit

| Verificação | Condição de Aprovação |
|-------------|----------------------|
| Pipeline parou se score baixo | Quando score < threshold, `03-search-result.md` NÃO existe — confirma que pipeline não avançou |
| Sintomas novos viraram tarefa | Se houve `symptom_unknown`, há entrada no `PROGRESS.md` para adicionar à taxonomia |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Clarificação (se necessária) | `../../tickets/<id>/02-clarification.md` | Markdown: perguntas estruturadas + seção `## Resposta` (preenchida pelo Bruno) |
| Ticket clarificado | `../../tickets/<id>/02-clarified.md` | Markdown com frontmatter (campos do passo 8) |
