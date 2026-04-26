# Stage 03: Design

Propor arquitetura ICM concreta para o alvo: silos, contratos L2, mapping de cada arquivo existente para destino.

## Inputs

| Fonte | Arquivo / Localização | Escopo | Por quê |
|-------|----------------------|--------|---------|
| Stage 01 | `../01-scan/output/file-inventory.json` | Completo | Lista de arquivos a mapear |
| Stage 02 | `../02-classify/output/triage-report.md` | Versão pós-checkpoint | Modos confirmados + triage 60/30/10 |
| Stage 02 | `../02-classify/output/work-modes.md` | Completo | Sugestões iniciais de silo |
| Config | `../../_config/silo-archetypes.md` | Carregar só os arquétipos relevantes aos modos detectados | Catálogo de silos |
| Config | `../../_config/anti-patterns.md` | Seções 1-7 | Auditar design contra erros comuns |
| Config | `../../_config/icm-rubric.md` | Seção "Critérios para decidir camada" | Atribuir L0-L4 a cada arquivo |

## Process

1. Para cada modo confirmado, escolher arquétipo de silo (ver silo-archetypes). Não inventar silos sem modo.
2. Nomear silos (preferir nomes do arquétipo; renomear se domínio pedir).
3. Redigir contrato L2 preliminar de cada silo (Inputs/Process/Outputs/Layer Distribution).
4. Mapear cada arquivo do inventário para `(silo, layer, destination_path, confidence)`. Aplicar critérios de camada da icm-rubric.
5. `confidence <70` → `manual-review`. Arquivos em `.gitignore` → `action=leave`.
6. Detectar conflitos com ICM pré-existente do alvo. Marcar com resolução proposta.
7. Auditar design contra anti-patterns.md.
8. Escrever os 3 outputs.

## Checkpoints

Sem gate humano interno (próximo é após stage 04). Listar trade-offs em `proposed-architecture.md`.

## Audit

| Verificação | Aprovação |
|-------------|-----------|
| Cada silo tem ≥1 arquivo | Sem silo vazio |
| Cada arquivo do inventário no CSV | 1-para-1, exceto ignorados |
| CSV passa schema | Validar contra [file-mapping.schema.json](../../shared/schemas/file-mapping.schema.json) |
| Conflitos ICM listados | Seção `## Conflitos` |
| Anti-patterns checklist passa | Itens finais de [anti-patterns.md](../../_config/anti-patterns.md) |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Arquitetura proposta | `output/proposed-architecture.md` | Doc: lista de silos, contrato L2 resumido de cada, trade-offs |
| Mapa de silos | `output/silo-map.md` | Tabela: silo → propósito → modos cobertos |
| Mapping de arquivos | `output/file-mapping.csv` | CSV conforme schema (autoridade para stage 05) |

## Hand-off

Stage 04 lê os 3 outputs. CSV é fonte de verdade — não duplicar movimentos em outro lugar.
