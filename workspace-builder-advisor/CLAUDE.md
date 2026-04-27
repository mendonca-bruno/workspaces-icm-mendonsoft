# Workspace Builder Advisor — Consultor do workspace-builder-v2

> **Este workspace NÃO executa o builder.** Produz artefatos `.md` (`advice-*.md`) que você copia/cola ao operar o `workspace-builder-v2`. É um companheiro, não um substituto.

Aplica princípios da **Interpreted Context Methodology** (ICM, paper arXiv 2603.16021v2) e da metodologia **ClifNotes** (60/30/10, L0-L4, voice 3-arquivos, stage-gate) para guiar o usuário em três momentos: (1) refinar a ideia antes do setup, (2) responder bem Q1-Q7 do builder, (3) tomar decisões certas em cada um dos 7 stages.

## Variável de instância

- `BUILDER_PATH` = caminho absoluto do `workspace-builder-v2`. Default: `c:\Users\bruno\Downloads\ClifNotes\Interpreted-Context-Methdology-main\workspaces\workspace-builder-v2\`. Coletado em `setup/questionnaire.md` e usado por `diagnose`.

## Mapa de Pastas

```
workspace-builder-advisor/
├── CLAUDE.md              (você está aqui)
├── CONTEXT.md             (roteamento por estado do usuário)
├── PROGRESS.md            (qual phase rodou, último output)
├── setup/
│   └── questionnaire.md   (3 perguntas: ideia + estado no builder + path)
├── phases/
│   ├── 00-idea-refinement/   (ideia bruta → brief estruturado)
│   └── 01-coaching/          (Q1-Q7 OU stage NN → recomendações)
└── references/               (L3 estável — factory)
    ├── icm-principles.md     (5 princípios do paper)
    ├── clif-rubric.md        (60/30/10, L0-L4, stage-gate, anti-patterns)
    ├── decision-points.md    (10 pontos críticos do builder + flow map)
    └── voice/                (3 arquivos: tom, formato, don't-list)
```

## Triggers

| Keyword | Ação |
|---------|------|
| `setup` ou `configurar` | Abre `setup/questionnaire.md` (3 perguntas iniciais) |
| `refinar` | Entra em `phases/00-idea-refinement/CONTEXT.md` → produz `advice-refined-brief.md` |
| `coachear setup` | Entra em `phases/01-coaching/` modo Q1-Q7 → produz `advice-q1-q7.md` |
| `coachear stage NN` | Entra em `phases/01-coaching/` modo stage NN → produz `advice-stage-NN.md` |
| `diagnose` | Escaneia `{BUILDER_PATH}/stages/*/output/` para detectar onde o usuário parou e roteia automaticamente |
| `status` | Lista quais `advice-*.md` já foram gerados em `phases/*/output/` |

### Como `diagnose` funciona

Para cada `{BUILDER_PATH}/stages/NN-name/output/`, verificar se existe arquivo além de `.gitkeep`:
- Nenhum stage com output → roteia para `refinar` (usuário ainda não começou)
- Stage 01 OK, Stage 02 vazio → roteia para `coachear stage 02`
- Stages 01-06 OK, Stage 07 vazio → roteia para `coachear stage 07`
- Tudo preenchido → "Builder completou. Revise `validation-report.md` no builder."

## Roteamento

| Tarefa | Ir Para |
|--------|---------|
| Refinar uma ideia bruta antes de rodar o builder | `phases/00-idea-refinement/CONTEXT.md` |
| Receber respostas recomendadas para Q1-Q7 do setup do builder | `phases/01-coaching/CONTEXT.md` (modo setup) |
| Coaching para um stage NN específico do builder | `phases/01-coaching/CONTEXT.md` (modo stage) |
| Entender os princípios ICM | `references/icm-principles.md` |
| Entender a rubrica Clif | `references/clif-rubric.md` |
| Ver os 10 pontos de decisão críticos do builder | `references/decision-points.md` |

## O Que Carregar

| Tarefa | Carregar | NÃO Carregar |
|--------|----------|--------------|
| Refinar ideia | `references/icm-principles.md`, `references/clif-rubric.md`, `references/voice/*` | `references/decision-points.md` (irrelevante antes do builder), playbook |
| Coachear setup (Q1-Q7) | `phases/01-coaching/playbook.md` (seções Q1-Q7), `references/decision-points.md`, `references/voice/*`, output de `00-idea-refinement` se existir | seções stage do playbook |
| Coachear stage NN | `phases/01-coaching/playbook.md` (seção stage NN), `references/decision-points.md`, `references/icm-principles.md`, `references/voice/*` | seções Q1-Q7 do playbook |
| Diagnose | Apenas estrutura de pastas em `{BUILDER_PATH}/stages/*/output/` | Nada do advisor além disso |

## Handoffs

- `phases/00-idea-refinement/output/advice-refined-brief.md` é input opcional para `phases/01-coaching/` modo setup.
- `phases/01-coaching/output/advice-q1-q7.md` é o que o usuário cola ao responder o `setup/questionnaire.md` do **builder**.
- `phases/01-coaching/output/advice-stage-NN.md` é o que o usuário lê antes de entrar em `{BUILDER_PATH}/stages/NN-*/CONTEXT.md`.

Cada output é um **edit surface**: o usuário pode modificar `advice-*.md` antes de levar ao builder.

## Out of Scope

Coisas que este workspace **deliberadamente não faz**:

- Não executa, modifica, nem inspeciona o conteúdo dos arquivos `output/*.md` do `workspace-builder-v2` (apenas detecta presença/ausência via `diagnose`).
- Não implementa integrações via hooks, scripts, MCP, APIs. Comunicação é 100% plain text via `advice-*.md`.
- Não gera o workspace final (esse é o job do builder).
- Não duplica `conventions-reference.md` do builder — referencia por path.
- Não toma decisões pelo usuário. Recomenda, justifica, alerta. A decisão é sempre do humano.
- Não cria dashboard ou camada de observabilidade. "Abra a pasta e leia" é suficiente (princípio ICM).
