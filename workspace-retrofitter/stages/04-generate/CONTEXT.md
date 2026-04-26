# Stage 04: Generate

Renderizar os arquivos ICM finais em pasta paralela `icm-overlay/` (fora do alvo). Não tocar no alvo.

## Inputs

| Fonte | Arquivo / Localização | Escopo | Por quê |
|-------|----------------------|--------|---------|
| Stage 03 | `../03-design/output/proposed-architecture.md` | Completo | Especificação dos silos |
| Stage 03 | `../03-design/output/silo-map.md` | Completo | Roteamento por silo |
| Stage 03 | `../03-design/output/file-mapping.csv` | Completo | Movimentos planejados (para gerar MIGRATION-PLAN) |
| Templates | `../../shared/templates/*` | Arquivos individuais por tipo | Esqueletos paramétricos |
| Config | `../../_config/icm-rubric.md` | Seção "Ratio de conteúdo" | Validar tamanhos finais |

## Process

1. Criar `<target>-icm-overlay/` (sibling do alvo). Se existir, sufixar `-<timestamp>`.
2. Renderizar `CLAUDE.md` raiz a partir do template. Validar ≤1.000 tok.
3. Renderizar `CONTEXT.md` raiz. Validar ≤500 tok.
4. Para cada silo: criar pasta no overlay, renderizar `CONTEXT.md` do silo. Validar ≤500 tok cada.
5. Para cada arquivo L3 do CSV: criar placeholder vazio no destino com cabeçalho.
6. Renderizar `MIGRATION-PLAN.md` com agregados do CSV.
7. Copiar `file-mapping.csv` de stage 03 para o overlay (esta é a versão editável).
8. Auditar tamanhos. Reduzir e re-renderizar se exceder.

## Checkpoints

Este stage prepara o gate #2. Instruir:

> "Inspecione `<target>-icm-overlay/`. Edite `file-mapping.csv` se necessário. Para autorizar: crie `APPROVED.md` no overlay e rode stage 05."

## Audit

| Verificação | Aprovação |
|-------------|-----------|
| Overlay fora do alvo | Caminho não começa com path do alvo |
| CLAUDE.md ≤1.000 tok | `word_count × 1.3` |
| CONTEXT.md raiz ≤500 tok | mesma fórmula |
| CONTEXT.md de silo ≤500 tok cada | mesma fórmula |
| MIGRATION-PLAN sem placeholders | Sem `{{...}}` residual |
| CSV válido | Passa [schema](../../shared/schemas/file-mapping.schema.json) |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Overlay completo | `<target>-icm-overlay/` (fora do alvo) | Diretório com CLAUDE.md, CONTEXT.md, silos, file-mapping.csv, MIGRATION-PLAN.md |
| Log de geração | `output/generation-log.md` | Markdown listando o que foi gerado e tamanhos finais |

## Hand-off

Stage 05 lê `file-mapping.csv` **do overlay** (não o de stage 03). Só roda com `APPROVED.md`.
