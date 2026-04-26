# Workspace Builder v2 — Code-Development Native

Este workspace guia a criação de novos workspaces ICM otimizados para **desenvolvimento de código**. Ele analisa codebases existentes, descobre workflows de desenvolvimento, e gera scaffolds completos com infraestrutura de testes, CI/CD, e verificação de contratos.

## Mapa de Pastas

```
workspace-builder-v2/
├── CLAUDE.md              (você está aqui)
├── CONTEXT.md             (comece aqui para roteamento de tarefas)
├── setup/
│   └── questionnaire.md   (onboarding -- perguntas sobre o projeto a construir)
├── stages/
│   ├── 01-analysis/       (analisar codebase/domínio existente)
│   ├── 02-discovery/      (entender o workflow de desenvolvimento)
│   ├── 03-mapping/        (formalizar contratos de stages com Verify)
│   ├── 04-scaffolding/    (gerar árvore de pastas e arquivos)
│   ├── 05-testing/        (gerar infraestrutura de testes e CI)
│   ├── 06-questionnaire/  (construir questionário de onboarding)
│   └── 07-validation/     (verificar contra convenções ICM + contratos)
└── references/
    ├── conventions-reference.md   (convenções MWP atualizadas)
    ├── code-patterns.md           (patterns para código)
    ├── examples/
    │   ├── script-to-animation-summary.md   (exemplo content workspace)
    │   ├── fullstack-api-summary.md         (exemplo fullstack dev)
    │   └── python-pipeline-summary.md       (exemplo data pipeline)
    └── templates/
        ├── claude-md-dev.md                 (template CLAUDE.md dev)
        ├── progress-template.md             (template PROGRESS.md)
        ├── ci-workflow-template.md          (template CI/CD)
        ├── stage-context-template.md        (template CONTEXT.md de stage)
        ├── workspace-claude-template.md     (template CLAUDE.md workspace)
        └── workspace-context-template.md    (template CONTEXT.md workspace)
```

## Triggers

| Keyword | Ação |
|---------|------|
| `setup` ou `configurar` | Executar onboarding -- perguntas sobre o projeto alvo |
| `analisar` | Iniciar análise de codebase existente (Stage 01) |
| `construir` | Iniciar pipeline completo de construção (Stage 01 → 07) |
| `status` | Mostrar progresso da pipeline para todos os 7 stages |

### Como `status` funciona

Escanear `stages/*/output/`. Para cada stage, se a pasta output contém arquivos (além de .gitkeep), o stage é COMPLETO. Caso contrário, PENDENTE. Renderizar:

```
Status da Pipeline: workspace-builder-v2

  [01-analysis] → [02-discovery] → [03-mapping] → [04-scaffolding] → [05-testing] → [06-questionnaire] → [07-validation]
     STATUS          STATUS          STATUS          STATUS            STATUS           STATUS               STATUS
```

## Roteamento

| Tarefa | Ir Para |
|--------|---------|
| Analisar codebase existente | `stages/01-analysis/CONTEXT.md` |
| Descobrir workflow de desenvolvimento | `stages/02-discovery/CONTEXT.md` |
| Formalizar contratos de stages | `stages/03-mapping/CONTEXT.md` |
| Gerar os arquivos do workspace | `stages/04-scaffolding/CONTEXT.md` |
| Gerar infraestrutura de testes | `stages/05-testing/CONTEXT.md` |
| Construir questionário de onboarding | `stages/06-questionnaire/CONTEXT.md` |
| Validar o workspace gerado | `stages/07-validation/CONTEXT.md` |

## O Que Carregar

| Tarefa | Carregar | NÃO Carregar |
|--------|----------|--------------|
| Analisar codebase | `references/code-patterns.md` | Todos os stages seguintes |
| Descobrir workflow | `stages/01-analysis/output/`, `references/conventions-reference.md`, `references/examples/` | `stages/03-mapping/` em diante |
| Formalizar contratos | `stages/02-discovery/output/`, `references/conventions-reference.md` | `references/examples/`, `stages/04-scaffolding/` em diante |
| Gerar scaffold | `stages/03-mapping/output/`, `references/conventions-reference.md`, `references/templates/*` | `stages/01-analysis/`, `stages/02-discovery/`, `references/examples/` |
| Gerar testes | `stages/01-analysis/output/`, workspace gerado em stage 04, `references/code-patterns.md` | `stages/02-discovery/`, `stages/03-mapping/` |
| Construir questionário | `stages/02-discovery/output/`, workspace gerado em stage 04, `references/templates/` | `stages/01-analysis/`, `stages/03-mapping/` |
| Validar workspace | Workspace gerado (stages 04+05), `stages/06-questionnaire/output/`, `references/conventions-reference.md` | `stages/01-analysis/`, `stages/02-discovery/`, `references/examples/` |

## Handoffs entre Stages

Cada stage escreve seu output na sua pasta `output/`. O próximo stage lê de lá. Se você editar um arquivo de output entre stages, o próximo stage pega suas edições.

O fluxo típico é sequencial (01 a 07), mas:
- Stage 05 (testing) também lê do stage 01 (analysis) diretamente
- Stage 07 (validation) lê dos stages 04, 05, e 06 diretamente
