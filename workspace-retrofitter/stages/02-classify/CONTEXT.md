# Stage 02: Classify

Diagnóstico: que tipo de workspace é este, que modos de trabalho contém, e onde cabe AI vs. infra.

## Inputs

| Fonte | Arquivo / Localização | Escopo | Por quê |
|-------|----------------------|--------|---------|
| Stage 01 | `../01-scan/output/file-inventory.json` | Completo | Distribuição de extensões, pastas |
| Stage 01 | `../01-scan/output/directory-tree.md` | Completo | Hierarquia para inferir modos |
| Stage 01 | `../01-scan/output/tech-manifests.json` | Completo | Detecção de stack |
| Config | `../../_config/detection-heuristics.md` | Completo | Regras de classificação |
| Config | `../../_config/60-30-10-triage.md` | Completo | Como aplicar triage por modo |

## Process

1. Aplicar tabela "Tipo macro" de detection-heuristics → `tipo`.
2. Aplicar tabela "Modos de trabalho" → modos com evidências.
3. Se `tipo ∈ {code, hybrid}`: detectar stack (linguagens, frameworks, tests, CI, linters).
4. Classificar maturidade com justificativa.
5. Para cada modo, aplicar 60/30/10. Listar `AI candidates to demote/adopt`.
6. Quantificar arquivos ambíguos (amostrar até 20).
7. **[Checkpoint]** Apresentar `triage-report.md` e aguardar confirmação/edição.
8. Escrever os 3 outputs.

## Checkpoints

| Após | Apresenta | Humano Decide |
|------|-----------|---------------|
| 7 | `triage-report.md` (tipo, modos, triage, ambíguos) | Reflete a realidade? |

## Audit

| Verificação | Condição de Aprovação |
|-------------|----------------------|
| Tipo macro atribuído | Exatamente um dos 5 valores |
| Modos detectados têm evidências | Cada modo lista ≥1 pasta/arquivo de suporte |
| Triage 60/30/10 cobre cada modo | Sem modo sem layer atribuído |
| Ambiguidade quantificada | Contagem absoluta + percentual |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Tipo de workspace | `output/workspace-type.md` | Markdown: tipo + maturidade + justificativa |
| Modos de trabalho | `output/work-modes.md` | Tabela: modo, evidências, sugestão de silo |
| Triage report | `output/triage-report.md` | Doc completo com seções de tipo, modos, 60/30/10, ambíguos |

## Hand-off

Stage 03 lê os 3 outputs. Edições humanas em `triage-report.md` são autoritativas.
