# Polyglot Porter — Roteamento de Tarefas

## O Que Este Workspace Faz

Recebe um workspace ICM em linguagem-fonte (Java/.NET/Python) e produz um workspace ICM equivalente em linguagem-alvo. Pipeline de 7 stages com 3 checkpoints humanos. Se o source não for ICM-compliant, o stage 00 invoca o retrofitter automaticamente.

---

## Para Onde Ir

| Você Quer... | Ir Para |
|--------------|---------|
| **Avaliar** ICM-compliance do source (pre-flight + auto-retrofitter) | `stages/00-preflight/CONTEXT.md` |
| **Inventariar** o source (módulos, frameworks, public surface) | `stages/01-source-analysis/CONTEXT.md` |
| **Escolher** o stack-flavor do target | `stages/02-target-discovery/CONTEXT.md` |
| **Mapear** padrões e idiomas (translation-map.csv) | `stages/03-pattern-mapping/CONTEXT.md` |
| **Gerar** o skeleton do target | `stages/04-scaffold-target/CONTEXT.md` |
| **Traduzir** o código file-by-file | `stages/05-translate/CONTEXT.md` |
| **Validar** a equivalência (build + test + diff) | `stages/06-validate-equivalence/CONTEXT.md` |
| **Configurar** restrições antes de começar | `setup/questionnaire.md` |

**Não leia tudo.** Identifique sua fase, carregue o CONTEXT.md daquele stage. CLAUDE.md tem o mapa completo.

---

## Fluxo da Pipeline

```
00-preflight  →  01-source-analysis  →  02-target-discovery  →  [Checkpoint #1]  →  03-pattern-mapping  →  [Checkpoint #2: APPROVED.md]  →  04-scaffold-target  →  05-translate  →  06-validate-equivalence  →  [Checkpoint #3]
 (gate)          (read inventory)       (pick stack)            target-decisions    (csv + idioms)         translation-map review            (skeleton)             (per-file)        (build+test+diff)             rubric sign-off
```

---

## Recursos Compartilhados

| Recurso | Localização | Quando Carregar |
|---------|-------------|-----------------|
| Pre-flight policy | `_config/preflight-policy.md` | Stage 00 |
| Language archetypes | `_config/language-archetypes.md` | Stages 01 (source-side), 02 (target-side) |
| Pattern equivalences | `_config/pattern-equivalences/<source>-to-<target>.md` | Stages 03, 05 |
| Idiom mapping | `_config/idiom-mapping/*` | Stage 03 |
| Anti-patterns | `_config/anti-patterns-migration.md` | Stages 03, 05 |
| Equivalence rubric | `_config/equivalence-rubric.md` | Stages 02 (entender níveis), 06 (avaliar) |
| Templates | `shared/templates/<target-lang>/` | Stage 04 |
| Translation-map schema | `shared/schemas/translation-map.schema.json` | Stages 03 (validar), 05 (re-validar) |
| Golden examples | `references/golden-examples/` | Stages 02 (cross-check), 05 (oracle), 06 (shape comparison) |

---

## Checkpoints Humanos

| Quando | Onde | O Que Aprovar |
|--------|------|---------------|
| Após stage 02 | `stages/02-target-discovery/output/target-decisions.md` | Stack-flavor (Spring Boot 3 → ASP.NET 8, ORM, DI) reflete o desejo? |
| Após stage 03 | `stages/03-pattern-mapping/output/translation-map.csv` + `mapping-report.md` | Mapeamento por arquivo está correto? Criar `APPROVED.md` em `stages/03-pattern-mapping/output/` para liberar geração. |
| Após stage 06 | `stages/06-validate-equivalence/output/equivalence-report.md` | Aceitar nível atingido (L1/L2/L3/L4 da rubric)? |

---

## Pares MVP

| Par | Status | Stack típico |
|-----|--------|--------------|
| Java ↔ .NET | MVP | Spring Boot 3 ↔ ASP.NET 8 + EF Core |
| Java ↔ Python | MVP | Spring Boot 3 ↔ FastAPI/Django + SQLAlchemy |
| .NET ↔ Python | Fase 2 | — |
