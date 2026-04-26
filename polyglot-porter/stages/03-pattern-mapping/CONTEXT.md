# Stage 03: Pattern Mapping

Construir o `translation-map.csv` — contrato de tradução por arquivo/classe com idiom decisions explícitas. Termina em **checkpoint humano #2** (gated por `APPROVED.md`).

## Inputs

| Fonte | Arquivo / Localização | Escopo | Por quê |
|-------|----------------------|--------|---------|
| Stage 01 | `../01-source-analysis/output/source-inventory.md` | Completo | Lista de arquivos a mapear |
| Stage 01 | `../01-source-analysis/output/surface-map.json` | Completo | Garantir 1:1 surface coverage |
| Stage 02 | `../02-target-discovery/output/target-decisions.md` | Versão pós-checkpoint #1 | Stack-flavor confirmado |
| Stage 02 | `../02-target-discovery/output/dependency-map.json` | Completo | Equivalências de libs |
| Config | `../../_config/pattern-equivalences/<source>-to-<target>.md` | Carregar apenas o par direcional ativo | Tabela canônica |
| Config | `../../_config/idiom-mapping/null-and-optional.md` | Completo | Decisões null/optional |
| Config | `../../_config/idiom-mapping/error-model.md` | Completo | Decisões error |
| Config | `../../_config/idiom-mapping/concurrency.md` | Completo | Decisões concurrency |
| Config | `../../_config/anti-patterns-migration.md` | Completo | Auditoria final |
| Schema | `../../shared/schemas/translation-map.schema.json` | Completo | Validação do CSV |

## Process

1. Para cada arquivo no `source-inventory.md` que esteja no scope, criar entrada no CSV com colunas (ver schema):
   - `source_path`, `source_type` (controller/service/etc.), `target_path`, `target_type`
   - `target_class_name`, `target_namespace_or_package`
   - `idiom_decision` (literal | refactor-to-target-idiom | combine-multiple | split | skip)
   - `null_strategy` (optional | nullable | not-applicable)
   - `error_strategy` (exception | result-type | http-status-only)
   - `concurrency_strategy` (sync | async-io | cpu-bound | thread-pool)
   - `transactional` (yes | no | n/a)
   - `relationship_strategy` (apenas para entities: cascade-all | manual | etc.)
   - `confidence` (0–100)
   - `notes`
2. Aplicar `pattern-equivalences/*` linha a linha para mapear anotações/atributos/decoradores.
3. Aplicar `idiom-mapping/*` para preencher campos de estratégia.
4. Para cada arquivo do `surface-map.json`, garantir que existe entrada CSV com `target_path` ou `action=skip` justificado.
5. `confidence < 70` → marcar `manual-review` em notes; será destacado no `mapping-report.md`.
6. Validar CSV contra schema (`shared/schemas/translation-map.schema.json`).
7. Auditar contra `anti-patterns-migration.md` checklist final.
8. Escrever `mapping-report.md` resumindo:
   - Total de entradas, breakdown por `idiom_decision`
   - Lista de `manual-review`
   - Anti-pattern audit results
   - Trade-offs significativos
9. Escrever instruções claras no topo do `mapping-report.md`: usuário deve revisar CSV, editar livremente, criar `APPROVED.md` na pasta `output/` para liberar stage 04.

## Checkpoints

| Quando | Onde | Aprovação |
|---|---|---|
| Após geração do CSV + report | `output/translation-map.csv` + `output/mapping-report.md` | Usuário revisa, edita. Cria `output/APPROVED.md` (qualquer conteúdo, marcador de autorização). Sem este arquivo, stage 04 recusa rodar. |

## Audit

| Verificação | Aprovação |
|-------------|-----------|
| CSV valida schema | parse OK contra `translation-map.schema.json` |
| Cobertura: cada entrada de `surface-map.json` tem destino | Sem orphans |
| `idiom_decision` preenchido em todas as linhas | Sem vazio |
| Anti-pattern checklist | Todos os itens em `anti-patterns-migration.md` § "Checklist final" passando |
| Confidence < 70 destacado | Lista em `mapping-report.md` § "Manual Review" |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Mapa de tradução | `output/translation-map.csv` | CSV conforme schema (autoridade para stages 04+05) |
| Relatório de mapeamento | `output/mapping-report.md` | MD: estatísticas + manual-review + audit + trade-offs |
| Autorização | `output/APPROVED.md` | Criado pelo usuário após revisão |

## Hand-off

Stage 04 verifica presença de `APPROVED.md` antes de rodar. Lê CSV (autoridade) + `shared/templates/<target-lang>/`.
