# Patterns de Código

Referência de patterns comuns para desenvolvimento de código. Usado pelo Stage 01 (Análise) para detectar patterns existentes e pelo Stage 05 (Testing) para gerar infraestrutura.

---

## Workflows de Desenvolvimento

### Feature Branch Flow
```
main ← feature/[nome] ← commits
PR → review → merge → deploy
```
- Branches curtas (1-3 dias)
- PR obrigatório para merge
- CI roda em cada PR
- Deploy automático após merge em main

### Trunk-Based Development
```
main ← commits diretos (com feature flags)
Deploy contínuo
```
- Commits diretos em main
- Feature flags para funcionalidades incompletas
- Deploy contínuo
- Rollback rápido

### Gitflow
```
main ← release ← develop ← feature
hotfix → main + develop
```
- Branches de release para versões
- Develop como branch de integração
- Hotfix direto em main

---

## Patterns de Stage para Código

### Pipeline de Desenvolvimento (Genérico)
```
Planning → Contracts → Implementation → Testing → Deployment
  (o quê)    (como)       (código)       (verifica)  (entrega)
```

### Pipeline de API Backend
```
01-planning/     → Requisitos, endpoints, models
02-contracts/    → OpenAPI spec, schemas, types
03-implementation/ → Routes, services, repositories
04-testing/      → Unit, integration, e2e
05-deployment/   → Docker, CI/CD, infra
```

### Pipeline de Frontend
```
01-design/       → Wireframes, design system tokens
02-components/   → Component library
03-pages/        → Page composition, routing
04-integration/  → API integration, state
05-testing/      → Unit, visual regression, e2e
```

### Pipeline de Data
```
01-ingestion/    → Sources, extraction
02-transformation/ → Cleaning, normalization
03-analysis/     → Queries, aggregation
04-reporting/    → Dashboards, exports
```

---

## Patterns de Teste por Stack

### Python (pytest)
```
tests/
├── conftest.py           # Fixtures compartilhadas
├── test_[module].py      # Testes unitários
├── integration/
│   └── test_[flow].py    # Testes de integração
└── e2e/
    └── test_[scenario].py # Testes end-to-end
```
- **Runner:** pytest
- **Cobertura:** pytest-cov
- **Mocks:** unittest.mock, pytest-mock
- **Fixtures:** conftest.py para shared, inline para específicas
- **Naming:** `test_[funcao]_[cenario]_[resultado_esperado]`
- **Comando:** `pytest -v --cov=src`

### JavaScript/TypeScript (Jest/Vitest)
```
src/
├── [module]/
│   ├── index.ts
│   └── __tests__/
│       └── index.test.ts  # Colocated tests
tests/
├── integration/
│   └── [flow].test.ts
└── e2e/
    └── [scenario].spec.ts
```
- **Runner:** vitest (moderno) ou jest
- **Cobertura:** c8 ou istanbul
- **Mocks:** vi.mock() ou jest.mock()
- **Naming:** `describe('[módulo]') → it('deve [comportamento]')`
- **Comando:** `npm test` ou `npx vitest`

### Go (go test)
```
[package]/
├── handler.go
├── handler_test.go       # Colocated tests
└── testdata/
    └── fixture.json      # Test data
```
- **Runner:** go test
- **Cobertura:** go test -cover
- **Mocks:** interfaces + structs de teste
- **Naming:** `Test[Função]_[cenário]`
- **Comando:** `go test ./... -v -cover`

---

## Patterns de CI por Plataforma

### GitHub Actions
```yaml
# .github/workflows/verify.yml
name: Verificação
on: [push, pull_request]
jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup
        # Setup runtime (Node, Python, Go, etc.)
      - name: Install
        # Install dependencies
      - name: Lint
        # Run linter
      - name: Type Check
        # Run type checker (se aplicável)
      - name: Test
        # Run tests
      - name: Build
        # Build (se aplicável)
```

### GitLab CI
```yaml
# .gitlab-ci.yml
stages: [lint, test, build]
lint:
  stage: lint
  script: [comando de lint]
test:
  stage: test
  script: [comando de teste]
build:
  stage: build
  script: [comando de build]
```

### Local-Only (Makefile)
```makefile
.PHONY: lint test build verify
lint:
	[comando de lint]
test:
	[comando de teste]
build:
	[comando de build]
verify: lint test build
```

---

## Patterns de Contrato/Schema

### TypeScript Interfaces (Shared Types)
```typescript
// shared/types/order.ts
export interface Order {
  id: string;
  status: OrderStatus;
  items: OrderItem[];
  createdAt: Date;
}
```

### Pydantic Models (Python)
```python
# shared/schemas/order.py
from pydantic import BaseModel
class OrderSchema(BaseModel):
    id: str
    status: str
    items: list[OrderItemSchema]
    created_at: datetime
```

### OpenAPI Spec
```yaml
# shared/schemas/api.yml
paths:
  /orders:
    get:
      summary: Listar pedidos
      responses:
        '200':
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Order'
```

---

## Patterns de State Tracking

### PROGRESS.md
Arquivo no root que persiste estado entre sessões. Template em `references/templates/progress-template.md`.

### DECISIONS.md (ADR Simplificado)
```markdown
## [Data] — [Título da Decisão]
**Contexto:** [Por que essa decisão foi necessária]
**Decisão:** [O que foi decidido]
**Consequências:** [O que muda por causa disso]
```

### Changelog (CHANGELOG.md)
```markdown
## [Data] — v[versão]
### Adicionado
- [Feature nova]
### Modificado
- [Mudança em feature existente]
### Corrigido
- [Bug fix]
```
