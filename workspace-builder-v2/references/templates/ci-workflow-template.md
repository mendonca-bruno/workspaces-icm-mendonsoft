# Template de CI Workflow

Use como base para gerar configuração de CI. Adapte para a plataforma e stack do projeto.

---

## GitHub Actions — Python

```yaml
name: Verificação

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  verify:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:{{POSTGRES_VERSION}}
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test_db
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '{{PYTHON_VERSION}}'
      
      - name: Instalar dependências
        run: |
          pip install -e ".[dev]"
      
      - name: Lint
        run: |
          ruff check {{SRC_DIR}}/ {{TEST_DIR}}/
          ruff format --check {{SRC_DIR}}/ {{TEST_DIR}}/
      
      - name: Type Check
        run: |
          mypy {{SRC_DIR}}/
      
      - name: Testes
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test_db
        run: |
          pytest -v --cov={{SRC_DIR}} --cov-fail-under={{MIN_COVERAGE}}
      
      - name: Build Docker
        run: |
          docker build -t {{PROJECT_NAME}}:test .
```

---

## GitHub Actions — Node.js/TypeScript

```yaml
name: Verificação

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  verify:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [{{NODE_VERSION}}]

    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Instalar dependências
        run: npm ci
      
      - name: Lint
        run: npm run lint
      
      - name: Type Check
        run: npm run typecheck
      
      - name: Testes
        run: npm test -- --coverage --coverageThreshold='{"global":{"branches":{{MIN_COVERAGE}},"functions":{{MIN_COVERAGE}},"lines":{{MIN_COVERAGE}}}}'
      
      - name: Build
        run: npm run build
```

---

## Local-Only (Makefile)

```makefile
.PHONY: lint test build verify clean

lint:
	{{LINT_CMD}}

typecheck:
	{{TYPECHECK_CMD}}

test:
	{{TEST_CMD}}

build:
	{{BUILD_CMD}}

verify: lint typecheck test build
	@echo "✅ Todas as verificações passaram"

clean:
	{{CLEAN_CMD}}
```

---

## Script de Verificação por Stage

```bash
#!/usr/bin/env bash
# _scripts/verify-{{STAGE_NAME}}.sh
# Verificações automatizadas para o stage {{STAGE_NAME}}

set -euo pipefail

echo "🔍 Verificando stage: {{STAGE_NAME}}"

# Verificação 1: {{CHECK_1_NAME}}
echo "  → {{CHECK_1_NAME}}..."
{{CHECK_1_CMD}}

# Verificação 2: {{CHECK_2_NAME}}
echo "  → {{CHECK_2_NAME}}..."
{{CHECK_2_CMD}}

echo "✅ Stage {{STAGE_NAME}} verificado com sucesso"
```

---

## Script Run-All-Checks

```bash
#!/usr/bin/env bash
# _scripts/run-all-checks.sh
# Executa todas as verificações em sequência

set -euo pipefail

FAIL_FAST=${1:-"--fail-fast"}
FAILED=0

run_check() {
    local script=$1
    local name=$2
    echo ""
    echo "━━━ $name ━━━"
    if bash "$script"; then
        echo "✅ $name: PASS"
    else
        echo "❌ $name: FAIL"
        FAILED=$((FAILED + 1))
        if [ "$FAIL_FAST" = "--fail-fast" ]; then
            echo "Abortando (--fail-fast). Use --continue para rodar todos."
            exit 1
        fi
    fi
}

{{VERIFY_SCRIPT_CALLS}}

echo ""
echo "━━━━━━━━━━━━━━━━━━"
if [ $FAILED -eq 0 ]; then
    echo "✅ Todas as verificações passaram"
else
    echo "❌ $FAILED verificação(ões) falhou(aram)"
    exit 1
fi
```
