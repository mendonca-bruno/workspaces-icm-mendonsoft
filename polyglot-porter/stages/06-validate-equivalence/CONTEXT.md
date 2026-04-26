# Stage 06: Validate Equivalence

Avaliar overlay contra os 4 níveis da `equivalence-rubric.md`. Build, rodar testes, contract-diff. Termina em **checkpoint humano #3** (sign-off final).

## Inputs

| Fonte | Arquivo / Localização | Escopo | Por quê |
|-------|----------------------|--------|---------|
| Stage 05 | `<overlay>/` | Tree completo | O target a avaliar |
| Stage 03 | `../03-pattern-mapping/output/translation-map.csv` | Completo | Para auditoria de surface coverage (L4) |
| Stage 01 | `../01-source-analysis/output/surface-map.json` | Completo | 100% mapping check (L4) |
| Source | `<source_path>` (test corpus) | Conforme `test_paths` | Para rodar source tests + extrair fixtures |
| Config | `../../_config/equivalence-rubric.md` | Completo | Critérios e comandos de cada nível |
| References | `../../references/golden-examples/` (apenas o arquivo do target: `spring-petclinic.md` / `eshop-on-web.md` / `full-stack-fastapi-template.md`) | Seção "Expected shape" | Cross-check de layout |

## Process

### L1 — Compiles

1. Rodar comando de build do target conforme rubric (`mvnw package` / `dotnet build` / `pip install -e .`).
2. Capturar stdout/stderr em `build-log.txt`.
3. Se falha: registrar em `equivalence-report.md` § L1, listar erros e arquivos afetados. **Parar** — níveis seguintes inviáveis.

### L2 — Tests Pass

4. Rodar suite de testes do target (`mvnw test` / `dotnet test` / `pytest`).
5. Coletar resultados em `test-report.xml` + sumário em `test-summary.md`.
6. Se algum teste falha: registrar em `equivalence-report.md` § L2.

### L3 — Behavior Parity (opcional, se solicitado)

7. Identificar N endpoints/operações representativas do `surface-map.json` (mínimo 5).
8. Para cada um, gravar request fixtures contra source (HTTP requests com expected responses).
9. Subir target localmente; replay das fixtures; comparar response (status + body + headers).
10. Tolerâncias: ler `equivalence-fixtures.yaml` (criado pelo usuário se necessário).
11. Diff por caso em `output/diffs/<case-id>.diff`.

### L4 — Coverage & Surface Parity (opcional)

12. Rodar coverage tool em source e target.
13. Diff em `coverage-diff.md`.
14. Cruzar `surface-map.json` × `translation-map.csv`. Listar não-mapeados em `unmapped.md`.

### Sign-off

15. Compilar `equivalence-report.md`:
    - Resumo: nível atingido vs nível-alvo
    - Para cada nível, PASS/FAIL com evidências (paths para `build-log.txt`, `test-report.xml`, etc.)
    - Lista de gaps
16. Instruir usuário no topo: revisar relatório, decidir aceitação. Sem `SIGNED-OFF.md`, migração fica em estado "pending review".

## Checkpoints

| Quando | Onde | Aprovação |
|---|---|---|
| Após geração do equivalence-report | `output/equivalence-report.md` | Usuário aceita nível atingido criando `output/SIGNED-OFF.md`. Pode escolher rejeitar e voltar para stage 03 ou 05 com correções. |

## Audit

| Verificação | Aprovação |
|-------------|-----------|
| Build do target executou | `build-log.txt` presente |
| L1 status registrado | PASS ou FAIL explícito |
| L2 status registrado (se L1 PASS) | idem |
| Gaps listados (se algum nível FAIL) | Seção dedicada em report |
| Sign-off file presente para "complete" | `SIGNED-OFF.md` ou `equivalence-report.md` indica `STATUS: REJECTED` |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Relatório de equivalência | `output/equivalence-report.md` | MD: nível atingido + evidências + gaps |
| Build log | `output/build-log.txt` | stdout/stderr do build |
| Test report | `output/test-report.xml` | JUnit ou equivalente |
| Test summary | `output/test-summary.md` | MD: agregado |
| Coverage diff (L4) | `output/coverage-diff.md` | MD: source vs target |
| Surface coverage (L4) | `output/surface-coverage.md` | MD: 1:1 audit |
| Unmapped (L4) | `output/unmapped.md` | MD: itens sem destino |
| Diffs por caso (L3) | `output/diffs/<case-id>.diff` | Unified diff |
| Sign-off (opcional, criado pelo user) | `output/SIGNED-OFF.md` | Marcador de aceite |

## Hand-off

Pipeline termina aqui. Overlay em `<overlay>/` é o produto entregue. Rollback = `rm -rf <overlay>` (source nunca foi modificado por stages 01–06; apenas stage 00, e só via retrofitter com seu próprio rollback).
