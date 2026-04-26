# Workspace Retrofitter — Adapt Existing Directory to ICM

Este workspace recebe um **diretório arbitrário** do usuário (repo de código, pasta de conteúdo, mistura caótica) e o reestrutura segundo a [Interpreted Context Methodology](../../arXiv-2603.16021v2/ICM__1_.tex), o framework 60/30/10 e o padrão de camadas L0-L4. Todas as modificações no alvo passam por checkpoint humano explícito.

**Diferença vs. outros builders:** [workspace-builder](../workspace-builder/), [workspace-builder-v2](../workspace-builder-v2/) e [workspace-builder-advisor](../workspace-builder-advisor/) criam workspaces do zero. O retrofitter **adapta um diretório existente**.

## Mapa de Pastas

```
workspace-retrofitter/
├── CLAUDE.md              ← você está aqui
├── CONTEXT.md             ← roteamento das 5 fases
├── README.md              ← como usar (input, fluxo)
├── _config/               ← L3 persistente do retrofitter
│   ├── icm-rubric.md
│   ├── silo-archetypes.md
│   ├── detection-heuristics.md
│   ├── 60-30-10-triage.md
│   └── anti-patterns.md
├── stages/
│   ├── 01-scan/           ← read-only do alvo
│   ├── 02-classify/       ← diagnóstico (tipo, modos, triage)
│   ├── 03-design/         ← propor silos + file mapping
│   ├── 04-generate/       ← renderizar overlay (fora do alvo)
│   └── 05-apply/          ← mover arquivos (gated por APPROVED.md)
├── shared/
│   ├── templates/         ← esqueletos paramétricos
│   └── schemas/           ← schema do file-mapping.csv
└── setup/
    └── questionnaire.md   ← perguntas mínimas antes de stage 01
```

## Triggers

| Keyword | Ação |
|---------|------|
| `setup` | Abrir questionnaire e coletar restrições do usuário |
| `scan --target=<path>` | Iniciar pipeline em stage 01 |
| `status` | Mostrar progresso por stage (presença de outputs) |
| `apply` | Executar stage 05 (exige APPROVED.md no overlay) |
| `rollback` | Reverter stage 05 usando rollback-manifest.json |

## Roteamento

| Tarefa | Ir Para |
|--------|---------|
| Escanear o diretório alvo | `stages/01-scan/CONTEXT.md` |
| Classificar tipo e modos de trabalho | `stages/02-classify/CONTEXT.md` |
| Projetar silos e mapping | `stages/03-design/CONTEXT.md` |
| Gerar overlay com arquivos ICM | `stages/04-generate/CONTEXT.md` |
| Aplicar migração in-place | `stages/05-apply/CONTEXT.md` |
| Reverter aplicação | `stages/05-apply/CONTEXT.md` (modo --rollback) |

## O Que Carregar

| Tarefa | Carregar | NÃO Carregar |
|--------|----------|--------------|
| Stage 01 (scan) | `setup/questionnaire.md` (respostas) | qualquer coisa em `_config/` ou stages seguintes |
| Stage 02 (classify) | `_config/detection-heuristics.md`, `_config/60-30-10-triage.md`, outputs de stage 01 | `_config/icm-rubric.md`, templates |
| Stage 03 (design) | `_config/silo-archetypes.md`, `_config/anti-patterns.md`, `_config/icm-rubric.md` (seção camadas), outputs de stage 02 | `_config/detection-heuristics.md`, templates de geração |
| Stage 04 (generate) | `shared/templates/*`, `_config/icm-rubric.md` (seção ratio), outputs de stage 03 | `_config/detection-heuristics.md`, `_config/60-30-10-triage.md` |
| Stage 05 (apply) | `shared/schemas/file-mapping.schema.json`, overlay (CLAUDE.md, CONTEXT.md, file-mapping.csv) | qualquer coisa em `_config/` |

## Convenções

- **Caminho do alvo:** sempre absoluto, passado via `--target=<path>`. Nunca inferir.
- **Overlay:** criado em `<target>-icm-overlay/` (sibling do alvo, **fora**). Se já existir, sufixar com timestamp.
- **CSV é fonte de verdade:** stage 05 lê `file-mapping.csv` do overlay (versão pós-edição do usuário), nunca o de stage 03.
- **Reversibilidade:** stage 05 escreve `rollback-manifest.json` antes de qualquer movimento.

## Hand-offs entre Stages

Sequencial 01 → 02 → 03 → 04 → 05. Cada stage lê apenas dos imediatamente anteriores. Edição manual de outputs entre stages é esperada (fluxo Unix: arquivo é a interface).

| Stage | Lê de | Escreve em |
|-------|-------|------------|
| 01 | `--target` | `stages/01-scan/output/` |
| 02 | stage 01 outputs | `stages/02-classify/output/` |
| 03 | stage 01-02 outputs | `stages/03-design/output/` |
| 04 | stage 03 outputs + templates | `<target>-icm-overlay/` + `stages/04-generate/output/` |
| 05 | overlay (gated por APPROVED.md) | alvo (in-place) + `stages/05-apply/output/` |
