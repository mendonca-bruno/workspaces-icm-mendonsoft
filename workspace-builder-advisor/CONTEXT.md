# Workspace Builder Advisor — Roteamento por Estado

## O Que Este Workspace Faz

Aconselha o usuário enquanto ele opera o `workspace-builder-v2`. Aplica princípios ICM (paper) e Clif (60/30/10, L0-L4, voice, stage-gate) em três momentos: refinar ideia, responder Q1-Q7, decidir bem em cada stage.

---

## Decisão Binária Inicial

Em qual estado você está agora?

| Estado | Sintoma | Ir Para |
|--------|---------|---------|
| **Tenho só uma ideia bruta** | Ainda não abri o builder. Não sei descrever em uma frase. Não sei se é greenfield. | `phases/00-idea-refinement/CONTEXT.md` |
| **Vou rodar o setup do builder** | Já tenho uma ideia clara. Vou começar a responder Q1-Q7 do builder. | `phases/01-coaching/CONTEXT.md` (modo setup) |
| **Estou num stage NN do builder** | Já passei pelo setup. Estou prestes a entrar em `{BUILDER_PATH}/stages/NN-*/CONTEXT.md`. | `phases/01-coaching/CONTEXT.md` (modo stage) |
| **Não sei onde estou** | Comecei o builder, parei, voltei dias depois. | Use o trigger `diagnose` no CLAUDE.md |
| **Builder terminou (Stage 07 OK)** | Já tenho `validation-report.md`. | Sem advisor — leia o relatório do builder. |

**Não leia tudo.** Identifique seu estado, carregue apenas o que precisa.

---

## Fluxo do Advisor

```
ideia bruta → 00-idea-refinement → advice-refined-brief.md
                                          ↓ (opcional)
{Q1-Q7 do builder} ←→ 01-coaching modo setup → advice-q1-q7.md
                                          ↓
{stage NN do builder} ←→ 01-coaching modo stage NN → advice-stage-NN.md
```

`00` é independente de `01`. `01` modo setup é independente de `01` modo stage. Sem ciclos. Você pode pular `00` se já tem ideia clara.

---

## Recursos Compartilhados

| Recurso | Localização | Quando Carregar |
|---------|-------------|-----------------|
| Princípios ICM (5) | `references/icm-principles.md` | Em qualquer phase |
| Rubrica Clif (60/30/10, L0-L4, stage-gate, anti-patterns) | `references/clif-rubric.md` | Em qualquer phase |
| 10 pontos críticos do builder | `references/decision-points.md` | `01-coaching` (qualquer modo) |
| Voice (3 arquivos) | `references/voice/` | Toda escrita de `advice-*.md` |
| Playbook consolidado (Q1-Q7 + stages 01-07) | `phases/01-coaching/playbook.md` | Apenas `01-coaching` |
| Conventions MWP (do builder) | `{BUILDER_PATH}/references/conventions-reference.md` | Citar, não duplicar |
