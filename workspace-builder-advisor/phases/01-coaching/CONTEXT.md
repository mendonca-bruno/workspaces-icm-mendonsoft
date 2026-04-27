# Phase 01: Coaching

Gera coaching contextual para um momento específico do `workspace-builder-v2`: ou o setup (Q1-Q7), ou um stage (01-07). Roteador interno por modo.

## Modos

| Modo | Trigger | Input | Output |
|------|---------|-------|--------|
| **setup** | `coachear setup` | (opcional) `../00-idea-refinement/output/advice-refined-brief.md` | `output/advice-q1-q7.md` |
| **stage NN** | `coachear stage NN` (NN = 01..07) | `playbook.md` seção stage NN | `output/advice-stage-NN.md` |
| **validation** | `coachear stage 07` (caso especial) | `playbook.md` seção stage 07 + DP-10 | `output/advice-stage-07.md` |

## Inputs (transversais)

| Fonte | Arquivo | Escopo | Razão |
|-------|---------|--------|-------|
| Local | `playbook.md` | seção do modo (Q1-Q7 OU stage NN) | conteúdo do coaching |
| Reference | `../../references/decision-points.md` | DPs relevantes ao modo | pontos críticos |
| Reference | `../../references/icm-principles.md` | ler completo | citação de princípio |
| Reference | `../../references/clif-rubric.md` | ler completo | citação de heurística |
| Reference | `../../references/voice/*` | ler completo | tom + formato + don't-list |

## Process (modo setup)

1. Ler `advice-refined-brief.md` se existir; senão pedir respostas brutas do usuário para Q1-Q7.
2. Para cada Q de Q1 a Q7, ler seção correspondente em `playbook.md`.
3. Para cada Q, gerar bloco no formato Claim/Reason/How (`voice/format-patterns.md` §"Formato advice-q1-q7.md").
4. Endereçar **explicitamente** DP-1 (Q2 ambíguo) e DP-2 (Q6 subestimado).
5. **[CHECKPOINT]** Apresentar rascunho. Aceitar edições do usuário.
6. Gravar `output/advice-q1-q7.md`.
7. Aplicar stage-gate.

## Process (modo stage)

1. Identificar o stage NN do builder em que o usuário está prestes a entrar.
2. Ler seção `Stage NN` em `playbook.md`.
3. Identificar quais DPs (de DP-3 a DP-10) se aplicam — mínimo 3.
4. Para cada DP aplicável, gerar bloco Claim/Reason/How (`voice/format-patterns.md` §"Formato advice-stage-NN.md").
5. Listar 1 anti-padrão a evitar neste stage (vindo de `clif-rubric.md` §6 ou de DPs).
6. **[CHECKPOINT]** Apresentar rascunho. Aceitar edições.
7. Gravar `output/advice-stage-NN.md`.
8. Aplicar stage-gate.

## Audit (modo setup)

| Check | Condição |
|-------|----------|
| 7 respostas | Q1 a Q7 presentes, cada uma com Claim/Reason/How |
| DP-1 endereçado | Q2 distingue greenfield vs existing com critério explícito |
| DP-2 endereçado | Q6 inclui alerta para não subestimar + comparativo com exemplos |
| Justificativas ancoradas | Cada Reason cita ICM #N ou Clif (60/30/10, L0-L4, etc.) ou DP-N |
| Voice respeitado | constraints.md §1-22 |

## Audit (modo stage)

| Check | Condição |
|-------|----------|
| 3+ DPs endereçados | Mínimo três pontos críticos relevantes ao stage |
| Cada recomendação ancorada | Princípio ICM ou Clif citado em cada Claim/Reason/How |
| 1 anti-padrão listado | Identificado e justificado (por que falha) |
| Ação concreta no checkpoint | Each How aponta passo específico do CONTEXT.md do stage NN do builder |
| Voice respeitado | constraints.md §1-22 |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Setup advice | `output/advice-q1-q7.md` | `voice/format-patterns.md` §"Formato advice-q1-q7.md" |
| Stage advice | `output/advice-stage-NN.md` | `voice/format-patterns.md` §"Formato advice-stage-NN.md" |

## Handoff

Usuário lê o `advice-*.md` e leva ao builder. No modo setup, copia respostas para o `setup/questionnaire.md` do builder. No modo stage, lê antes de entrar no `CONTEXT.md` do stage correspondente do builder.
