# Phase 00: Idea Refinement

Refina uma ideia bruta em um brief estruturado pronto para alimentar Q1-Q7 do `workspace-builder-v2`. Use **antes** de o usuário interagir com o builder.

## Inputs

| Fonte | Arquivo | Escopo | Razão |
|-------|---------|--------|-------|
| User | resposta A1 do `setup/questionnaire.md` | ideia bruta em 1-3 frases | ponto de partida |
| Reference | `../../references/icm-principles.md` | ler completo | rubrica de qualidade |
| Reference | `../../references/clif-rubric.md` | seção 60/30/10 | triagem do que precisa de IA |
| Reference | `../../references/voice/*` | ler completo | tom + formato + don't-list |

## Process

1. Ler ideia bruta do usuário.
2. **Aplicar interrogatório socrático** (não dar solução pronta):
   - "Qual é o problema central, em uma frase, sem mencionar tecnologia?"
   - "Quem usa? Quem mantém? Quem consome o output?"
   - "O que está dentro do escopo? O que está fora?"
3. **Aplicar triagem 60/30/10** (clif-rubric §1):
   - Liste 3-5 tarefas que o sistema fará.
   - Para cada uma, classifique: tradicional / rule-based / IA.
   - Se >50% acabarem em "IA", desafiar — provavelmente erro de classificação.
4. **Predizer Q1-Q7** com base no acima:
   - Q1: nome do projeto/domínio (deriva de "problema central").
   - Q2: greenfield ou existing? (perguntar diretamente; aplicar DP-1 se ambíguo).
   - Q3: workflow em 1 frase (extrair do escopo + tarefas).
   - Q4: tech stack (perguntar; se usuário não souber, sinalizar como blocker — não advinhe).
   - Q5: perfil do usuário (perguntar).
   - Q6: estimativa de # de stages (aplicar DP-2 — comparar com exemplos do builder).
   - Q7: CI/CD (perguntar; default razoável: GitHub Actions).
5. **[CHECKPOINT]** Apresentar o brief em rascunho. Pedir correções/edição. Aceitar mudanças do usuário (princípio ICM #4 — edit surface).
6. Gravar `output/advice-refined-brief.md` no formato definido em `voice/format-patterns.md`.
7. Aplicar stage-gate: brief só está pronto se os 4 itens da checklist passam.

## Audit

| Check | Condição |
|-------|----------|
| Problema explícito | Brief tem seção "Problema" com 1 frase, sem jargão técnico |
| Triagem 60/30/10 feita | As 3 categorias têm conteúdo; soma ~100% |
| Predição Q1-Q7 completa | 7 entradas presentes, cada uma com justificativa em 1 frase |
| Bifurcação Q2 explícita | Brief deixa claro se é greenfield ou existing, com path quando aplicável |
| Voice respeitado | Sem emoji, sem cheerleading, sem hedging (constraints.md §7-14) |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Refined brief | `output/advice-refined-brief.md` | Markdown, schema em `voice/format-patterns.md` §"Formato advice-refined-brief.md" |

## Handoff

`output/advice-refined-brief.md` é input opcional para `phases/01-coaching/` modo setup. O usuário também pode levar o brief diretamente ao `workspace-builder-v2/setup/questionnaire.md` e usá-lo como base para suas respostas.
