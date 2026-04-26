# Workspace Retrofitter — Task Router

## O Que Este Workspace Faz

Recebe um diretório arbitrário e o adapta para ser um workspace ICM ideal para agentes de IA. Pipeline de 5 stages com 2 checkpoints humanos.

---

## Para Onde Ir

| Você Quer... | Ir Para |
|--------------|---------|
| **Escanear** o diretório alvo (read-only) | `stages/01-scan/CONTEXT.md` |
| **Classificar** tipo e modos de trabalho | `stages/02-classify/CONTEXT.md` |
| **Projetar** silos e mapping | `stages/03-design/CONTEXT.md` |
| **Gerar** o overlay ICM (fora do alvo) | `stages/04-generate/CONTEXT.md` |
| **Aplicar** migração in-place | `stages/05-apply/CONTEXT.md` |
| **Reverter** uma aplicação | `stages/05-apply/CONTEXT.md` (modo `--rollback`) |
| **Configurar** restrições antes de começar | `setup/questionnaire.md` |

**Não leia tudo.** Identifique sua fase, carregue o CONTEXT.md daquele stage. CLAUDE.md tem o mapa completo.

---

## Fluxo da Pipeline

```
01-scan  →  02-classify  →  [Checkpoint #1]  →  03-design  →  04-generate  →  [Checkpoint #2: APPROVED.md]  →  05-apply
 (read)     (diagnose)       triage-report      (silos+CSV)   (overlay)        MIGRATION-PLAN review            (in-place + rollback)
```

---

## Recursos Compartilhados

| Recurso | Localização | Quando Carregar |
|---------|-------------|-----------------|
| Detection heuristics | `_config/detection-heuristics.md` | Stage 02 |
| 60/30/10 triage | `_config/60-30-10-triage.md` | Stage 02 |
| Silo archetypes | `_config/silo-archetypes.md` | Stage 03 |
| ICM rubric | `_config/icm-rubric.md` | Stage 03 (seção camadas), Stage 04 (seção ratio) |
| Anti-patterns | `_config/anti-patterns.md` | Stage 03, Stage 04 |
| Templates | `shared/templates/*` | Stage 04 |
| File mapping schema | `shared/schemas/file-mapping.schema.json` | Stage 03 (validar), Stage 05 (re-validar) |

---

## Checkpoints Humanos

| Quando | Onde | O Que Aprovar |
|--------|------|---------------|
| Após stage 02 | `stages/02-classify/output/triage-report.md` | Classificação reflete a realidade do trabalho? |
| Após stage 04 | `<target>-icm-overlay/MIGRATION-PLAN.md` + `file-mapping.csv` | Movimentos propostos estão corretos? Criar `APPROVED.md` para autorizar. |
