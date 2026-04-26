# Equivalence Rubric — Critérios de Migração Bem-Sucedida

4 níveis ascendentes. Stage 06 (validate-equivalence) avalia o overlay contra cada nível e gera artefatos de evidência. O usuário aceita ou rejeita o nível atingido no checkpoint #3.

## Níveis

### L1 — Compiles

**Gate:** target builds clean usando o build tool padrão do stack-flavor escolhido.

| Stack | Comando | Sucesso |
|---|---|---|
| Java/Maven | `./mvnw clean package -DskipTests` | exit 0, JAR gerado |
| Java/Gradle | `./gradlew assemble` | exit 0, JAR gerado |
| .NET | `dotnet build -c Release` | exit 0, sem warnings de erro |
| Python | `python -m py_compile $(find . -name '*.py')` + `pip install -e .` | exit 0 |

**Artefato:** `output/build-log.txt` com stdout/stderr completos do build.

### L2 — Tests Pass

**Gate:** suite de testes do target passa. A suite vem de duas fontes:

1. **Tests portados** — testes do source traduzidos pelo stage 05 (entrada `*_test_*` no `translation-map.csv`).
2. **Tests retidos** — se `target_flavor` define tests próprios (ex: testes de integração já existentes no template), também devem passar.

| Stack | Comando | Sucesso |
|---|---|---|
| Java/Maven | `./mvnw test` | exit 0, 0 failures |
| .NET | `dotnet test` | exit 0, todos passing |
| Python | `pytest` | exit 0, 0 failed |

**Artefato:** `output/test-report.xml` (formato JUnit ou equivalente) + `output/test-summary.md`.

### L3 — Behavior Parity

**Gate:** mesmas fixtures HTTP/dados → mesmas saídas no source e no target, em N golden cases (mínimo 5).

**Como medir:**
1. Identificar N endpoints/operações representativas (de `surface-map.json` do stage 01).
2. Para cada caso, gravar request fixture (HTTP request body, query params, headers) e expected response do source rodando.
3. Subir target localmente; replay das mesmas fixtures; comparar response (status code, body deserializado, headers relevantes).
4. Tolerâncias permitidas: ordem de campos em JSON, timestamps relativos, IDs auto-gerados (declarar no `equivalence-fixtures.yaml`).

**Artefato:** `output/equivalence-report.md` com:
- Tabela: caso → source response → target response → diff (PASS/FAIL/TOLERATED)
- Total: X/N PASS
- Diffs detalhados em `output/diffs/<case-id>.diff`

### L4 — Coverage & Surface Parity

**Gate:**
- Delta de cobertura de testes (source vs target) ≤ 5 pontos percentuais.
- Public surface 100% mapeada — toda classe/função pública do source tem destino em `translation-map.csv` com `action != skip`.

**Como medir:**
- Source coverage: rodar tool nativo (`jacoco`, `coverlet`, `coverage.py`) na suite original.
- Target coverage: mesmo, na suite portada.
- Surface mapping: cruzar `surface-map.json` (do stage 01) contra `translation-map.csv` (pós-checkpoint #2). Listar não-mapeados.

**Artefato:** `output/coverage-diff.md`, `output/surface-coverage.md`, `output/unmapped.md`.

## Aceitação

| Nível | Significado prático |
|---|---|
| L1 | Migração estrutural; compila mas comportamento não verificado. Útil para refactor manual posterior. |
| L2 | Equivalência funcional ao nível dos testes existentes. **Aceitação v1 do polyglot-porter.** |
| L3 | Equivalência comportamental verificada por contract testing. Recomendado para produção. |
| L4 | Equivalência completa. Necessário se o target vai substituir o source em produção sem janela de paralelismo. |

## Política de fallback

Se um nível N falha mas o usuário quer aceitar o nível N-1, registrar em `equivalence-report.md`:

- Que nível foi atingido
- Que nível foi rejeitado e o porquê (lista de gaps)
- Lista de itens em `output/unmapped.md` ou `output/diffs/` para acompanhamento manual

Não silenciar gaps. Tornar visíveis para o sign-off do checkpoint #3.
