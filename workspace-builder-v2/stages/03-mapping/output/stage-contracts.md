# Stage 03 — Contratos Formais dos Stages do Workspace `wallet-build`

**Data:** 2026-04-25
**Inputs deste stage:** `../02-discovery/output/workflow-map.md`
**Output deste documento:** consumido pelo Stage 04 (Scaffolding) do builder para gerar a árvore de arquivos do workspace `wallet-build`

**Convenção de nomes:**
- `wallet-build` = workspace ICM (mora em `D:\pntrsoft\workspaces\wallet-build\`). Segue padrão `<escopo>-<ação>` dos 6 workspaces existentes (`dispatch-engine-build`, `ordersmonit-web-migration`, etc.).
- `wallet` = codebase do produto (mora em `D:\pntrsoft\wallet\`, espelhando `dispatch-engine/` e `ordersmonit/`). Os 6 stages do workspace constroem este codebase.

Este documento formaliza os 6 stages do workspace `wallet-build` em contratos com Inputs/Processo/Checkpoints/Verify/Audit/Outputs, apresenta o grafo de dependência, classifica dependências por tipo (file/package/schema), declara fontes canônicas e verifica unidirecionalidade.

---

## 1. Diagrama de Dependência (DAG)

```
                ┌──────────────────────────┐
                │  Stage 1                 │
                │  Discovery & contract    │
                │  (Colaborativo)          │
                └─────────────┬────────────┘
                              │ contracts, schemas, glossário
                              ▼
                ┌──────────────────────────┐
                │  Stage 2                 │
                │  Wallet core + ingestão  │
                │  + publisher hook (cross)│
                │  (Review-gate)           │
                └─────────────┬────────────┘
                              │ models, ledger, ingestão funcionando
                              ▼
                ┌──────────────────────────┐
                │  Stage 3                 │
                │  API + auth              │
                │  (Review-gate)           │
                └─────┬───────────────┬────┘
                      │               │
       same FastAPI   │               │   API REST consumida via HTTP
       app + models   │               │
                      ▼               ▼
   ┌──────────────────────────┐   ┌──────────────────────────┐
   │  Stage 4                 │   │  Stage 5                 │
   │  Painel manager          │   │  PWA + marketplace       │
   │  (Auto + safety)         │   │  (Review-gate)           │
   └─────────────┬────────────┘   └──────────────────────────┘
                 │
                 │ rule engine + simulador
                 │ (modifica credit_engine de Stage 2 + UI no manager)
                 ▼
   ┌──────────────────────────┐
   │  Stage 6 (opcional)      │
   │  Promoções avançadas     │
   │  (Auto)                  │
   └──────────────────────────┘
```

Fluxo unidirecional. **Stage 5 não tem dependência de Stage 4** (paralelizáveis após Stage 3). **Stage 6 depende de Stage 4** (UI de regras + dry-run vivem no manager) e de Stage 2 (extensão do `credit_engine.py`); **NÃO depende de Stage 5** — a PWA continua mostrando "R$ X recebido" indiferente se foi flat ou regra composta.

**Distinção crítica entre arestas Stage 3 → Stage 4 e Stage 3 → Stage 5:**

- **Stage 3 → Stage 4** é compartilhamento de processo (mesma app FastAPI, mesmos models e services do `wallet/src/`). O manager **não** chama a API REST sobre HTTP — usa imports Python diretos (`from src.services.credit_engine import ...`). Isso é intencional para manter uma única lógica de crédito/redenção. **Não extrair o manager para serviço separado sem revisar cuidadosamente este pressuposto** — extrair forçaria duplicação ou criação de cliente HTTP interno que duplica complexidade.
- **Stage 3 → Stage 5** é consumo HTTP cross-process (PWA é processo separado, com cliente fetch tipado a partir do `api-spec.yaml`).

**Paralelização real:**

- 4 ↔ 5 paralelizáveis após 3 (sessões/devs diferentes).
- 5 ↔ 6 paralelizáveis após 4 (acadêmico para solo dev, mas o DAG não mente).

---

## 2. Mapa de Dependências por Tipo

### 2.1 File Dependencies (Stage N lê output do Stage N-1)

| Stage | Lê de | Arquivos consumidos |
|-------|-------|---------------------|
| 1 | (nenhum stage anterior) | `D:\pntrsoft\integration-contract\CONTRACT.md` v1.0.0; `D:\pntrsoft\dispatch-engine\src\models\route.py`, `driver.py`; `D:\pntrsoft\dispatch-engine\.ai-context\backend.md`; decisões fixadas/reservadas do workflow-map §2/§3 |
| 2 | Stage 1 | `01-discovery/output/domain-model.md`, `contract-extension.md`, `outbox-schema.sql` |
| 3 | Stage 2 | `02-core/output/` (models/, schemas/, services/) |
| 4 | Stages 1, 2, 3 | `03-api/output/openapi.yaml`, `01-discovery/output/domain-model.md`, schemas do Stage 2. **Compartilha mesma app FastAPI com Stage 3** — imports diretos, não HTTP. |
| 5 | Stages 1, 3 | `03-api/output/openapi.yaml`, `01-discovery/output/domain-model.md`. **Não depende de Stage 4** — catálogo de vouchers é consultado via API HTTP em runtime (não é build-time dependency). |
| 6 | Stages 1, 2, 4 (não 5) | rule engine herda schema do `wallet_ledger` (Stage 2), modifica `credit_engine.py` (Stage 2), painel herda safety-by-design e Jinja2 do manager (Stage 4). PWA não muda. |

### 2.2 Package Dependencies (compartilhadas entre stages)

| Pacote | Stages | Fonte |
|--------|--------|-------|
| Python 3.12 + FastAPI 0.115+ + SQLAlchemy 2.0 async + asyncpg + Alembic 1.14+ + Pydantic v2 + ruff 0.8+ | 2, 3, 4, 6 | `pyproject.toml` na raiz do wallet |
| `httpx` (cliente HTTP para validar auth_code contra dispatch-engine) | 3 | mesmo `pyproject.toml` |
| `itsdangerous` (sessão server-side) + `argon2-cffi` (password hashing) | 4 | mesmo `pyproject.toml` |
| `firebase-admin` (FCM próprio) | 5 backend ; 5 frontend usa `firebase` JS SDK | mesmo `pyproject.toml` + `pwa/package.json` |
| Node 20+ + React 18 + Vite 5 + TypeScript 5 strict + vitest + ESLint | 5 | `pwa/package.json` |
| `playwright` (E2E) | 4, 5 | `pyproject.toml` + `pwa/package.json` |
| `pytest` + `pytest-cov` + `pytest-asyncio` | 2, 3, 4, 6 | `pyproject.toml` |
| `spectral-cli` | 3 | global ou devDependency |
| `markdownlint-cli2`, `sqlfluff` | 1 | opcionais, instalação local |

### 2.3 Schema Dependencies (tipos/contratos cross-stage)

Aplicando Pattern 12 (shared types) — cada schema tem **uma casa canônica**, stages referenciam.

| Schema | Casa canônica | Definido em | Consumido por |
|--------|---------------|-------------|---------------|
| Glossário do domínio | `shared/domain/wallet-glossary.md` | Stage 1 | todos |
| Schema da outbox `route_events` | `shared/schemas/route-event.md` | Stage 1 | 1, 2; também publicado no `contract-extension.md` (extensão do contrato global) |
| Schemas das tabelas do wallet (ledger, processed_events, voucher, payment_cycle, expiry_alert, manager_audit, rule, rule_application) | `shared/schemas/wallet-tables.md` | Stage 1 (esqueleto) → Stage 2 (campos finais via Alembic) | 2, 3, 4, 6 |
| OpenAPI da API do wallet | `shared/schemas/api-spec.yaml` | Stage 3 | 3, 4, 5 |
| Extensão do integration-contract global | `shared/contracts/integration-contract-extension.md` (também copiada para `D:\pntrsoft\integration-contract\CONTRACT.md` v1.1.0 como fonte de verdade pntrsoft) | Stage 1 (rascunho) → Stage 2 (versão final v1.1.0 após publisher hook) | todos |
| Constantes (dia de pagamento, expiração, polling, TTL, bônus) | `shared/constants/payment-rules.md` | Stage 1 | 2, 4, 5 |
| Decisões arquiteturais (ADRs) | `shared/decisions/DECISIONS.md` | Stage 1 (G1–G7, Q5–Q14 + flags) → atualizado pelos demais quando uma decisão `pending-*` se confirma | todos |

**Regra de fonte canônica:** se um stage downstream precisa modificar um schema, a modificação acontece **na casa canônica** e gera nova versão. Stages downstream não duplicam definições.

### 2.4 Cross-projeto (fora do workspace `wallet`)

Pattern especial — Stage 2 modifica código do dispatch-engine. Tratado como dependência cross-projeto explícita:

| Arquivo modificado | Projeto | Origem da mudança |
|--------------------|---------|-------------------|
| `D:\pntrsoft\dispatch-engine\src\services\route_events_publisher.py` (novo) | dispatch-engine | Stage 2 do wallet |
| `D:\pntrsoft\dispatch-engine\src\routers\routes.py` (linhas 78 e 123 — chamadas ao publisher) | dispatch-engine | Stage 2 do wallet |
| `D:\pntrsoft\dispatch-engine\alembic\versions\NNNN_add_route_events_outbox.py` (nova migration) | dispatch-engine | Stage 2 do wallet |
| `D:\pntrsoft\integration-contract\CONTRACT.md` (v1.0.0 → v1.1.0) | integration-contract | Stage 1 do wallet (rascunho) → Stage 2 do wallet (versão final) |

**Skill `/cross-check` cobre estes arquivos.** PR do Stage 2 toca dois projetos — fluxo de revisão deve refletir.

---

## 3. Fontes Canônicas (declaração explícita)

| Fato | Casa canônica única |
|------|---------------------|
| Definição de domínio (saldo, ledger, evento, voucher, breakage, conversão, cut-off) | `shared/domain/wallet-glossary.md` |
| Schema da `route_events` (outbox no dispatch) | `shared/schemas/route-event.md` |
| Schema do `wallet_ledger` (double-entry) | `shared/schemas/wallet-tables.md` |
| Schema do `processed_events` | `shared/schemas/wallet-tables.md` |
| Schema do `voucher` e `voucher_redemption` | `shared/schemas/wallet-tables.md` |
| Schema do `payment_cycle` | `shared/schemas/wallet-tables.md` |
| Schema do `manager_audit` | `shared/schemas/wallet-tables.md` |
| Schema do `rule` e `rule_application` | `shared/schemas/wallet-tables.md` |
| API REST (paths, request/response) | `shared/schemas/api-spec.yaml` |
| Contrato de integração ordersmonit↔dispatch↔wallet | `D:\pntrsoft\integration-contract\CONTRACT.md` (v1.1.0 com extensão wallet) |
| Constantes de produto (`payment_day_of_week`, `balance_expiry_months`, `conversion_bonus_multiplier`, `polling_interval_seconds`, `outbox_ttl_days`) | `shared/constants/payment-rules.md` |
| ADRs e decisões | `shared/decisions/DECISIONS.md` |
| Variáveis (placeholders) | `setup/questionnaire.md` (preenche) → propagadas para configs |

Nenhum fato tem mais de uma casa. Stages downstream que precisem mencionar o fato fazem **referência** (link/cite), não cópia.

---

## 4. Contratos Formais por Stage

### Stage 1 — Discovery & contract

**Inputs**

| Fonte | Arquivo | Escopo | Razão |
|-------|---------|--------|-------|
| Builder Stage 02 | `02-discovery/output/workflow-map.md` (este workspace, ICM externo) | §2 (decisões fixadas), §3 (decisões reservadas), §6 (shared/), §7 (placeholders) | Base do que escrever |
| Externo | `D:\pntrsoft\integration-contract\CONTRACT.md` v1.0.0 | Arquivo completo | Convenção do contrato a estender |
| Externo | `D:\pntrsoft\dispatch-engine\src\models\route.py` e `driver.py` | Modelos | Schema da `Driver` snapshot e da `route_events` |
| Externo | `D:\pntrsoft\dispatch-engine\.ai-context\backend.md` | Mapa de routers/services | Confirmar pontos de transição (`routes.py:78` e `:123`) |
| Usuário | (conversa) | Decisões de produto que o workflow-map reservou (Q11/Q10/Q13 podem ter atualização) | Calibrar working assumptions |

**Processo**

1. Carregar workflow-map.md e ler §2 (decisões fixadas) e §3 (working assumptions).
2. Produzir `domain-model.md`: glossário + entidades + diagrama do ledger double-entry + mecânica de conversão saldo→voucher (com bônus 1.10x se Q11=c) + mecânica de breakage (Q13).
3. Produzir `outbox-schema.sql`: DDL da tabela `route_events` no schema do dispatch-engine. Campos: `event_id UUID PK DEFAULT gen_random_uuid()`, `route_id UUID NOT NULL`, `driver_id UUID NOT NULL`, `event_type VARCHAR(50) NOT NULL DEFAULT 'route.completed'`, `payload JSONB NOT NULL`, `created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()`, `consumed_at TIMESTAMPTZ NULL`. Índice `(consumed_at, created_at)` para o poller. Sem FK física para `routes.id` (mantém transação atômica via SQLAlchemy).
4. Produzir `contract-extension.md`: rascunho da v1.1.0 do contrato global. Adicionar Seção 9 (Wallet — consumidor da outbox), Seção 10 (Schema OutboxEvent), Seção 11 (Mecânica de crédito + idempotência via `event_id`), atualizar Seção 7 (Changelog v1.1.0), atualizar Seção 8 (Arquivos contract-sensitive). Manter convenção numerada e tom imperativo do contrato existente.
5. Esboçar `wallet-tables.md`: tabelas wallet com tipos e relações (PK, índices), sem migrations Alembic ainda (isso é Stage 2). Tabelas: `wallet_ledger`, `processed_events`, `voucher`, `voucher_redemption`, `payment_cycle`, `expiry_alert`, `manager_audit`, `manager_user`, `rule`, `rule_application`.
6. Esboçar `payment-rules.md` em `shared/constants/`: extrai das constantes de §7 do workflow-map.
7. Inicializar `DECISIONS.md` em `shared/decisions/` com G1–G7, Q5–Q14, status (`active` / `pending-*`).
8. Inicializar `wallet-glossary.md` em `shared/domain/`.
9. **[Checkpoint]** — Apresentar `domain-model.md` + `contract-extension.md` ao usuário. Atualizar working assumptions de Q11/Q10/Q13 se houve resposta nova.
10. Audit (§ Audit). Se algo falha, voltar.

**Checkpoints**

| Após Passo | Agente Apresenta | Humano Decide |
|------------|-------------------|---------------|
| 9 | `domain-model.md` (glossário + ledger + mecânica) e `contract-extension.md` v1.1.0 draft | Se contrato e domínio estão corretos para fechar e prosseguir para implementação no Stage 2 |

**Verify**

```bash
# _scripts/verify-stage-1.sh
markdownlint-cli2 'shared/contracts/*.md' 'shared/domain/*.md' 'stages/01-discovery/output/*.md'
sqlfluff lint stages/01-discovery/output/outbox-schema.sql --dialect postgres
# Auto-verify: contrato extendido tem seções 9, 10, 11 e changelog v1.1.0
test -n "$(grep -E '^## 9\.' shared/contracts/integration-contract-extension.md)"
test -n "$(grep -E '^## 10\.' shared/contracts/integration-contract-extension.md)"
test -n "$(grep -E 'v1\.1\.0' shared/contracts/integration-contract-extension.md)"
```

**Audit**

| Verificação | Pass condition |
|-------------|----------------|
| Convenção do contrato preservada | Numeração contínua (1–11), seção "Mudanças breaking" presente em §9 e §10, changelog atualizado, lista de "Arquivos contract-sensitive" inclui novos arquivos do dispatch e do wallet |
| `event_id` definido | `route_events.event_id UUID PK` com `DEFAULT gen_random_uuid()` |
| Transição → outbox documentada | `contract-extension.md` cita `routes.py:78` (manual) e `:123` (auto-complete) como pontos de inserção |
| Idempotência declarada | Documentada na §11 com chave única `event_id` no `processed_events` |
| Glossário cobre termos críticos | saldo, ledger, evento, voucher, breakage, conversão, cut-off, double-entry presentes |
| `DECISIONS.md` registra working assumptions | Q11, Q10, Q13 com flag `pending-*` correto |

**Outputs**

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Modelo de domínio | `output/domain-model.md` | Markdown estruturado: glossário + entidades + diagramas |
| DDL da outbox | `output/outbox-schema.sql` | SQL Postgres |
| Extensão do contrato | `output/contract-extension.md` (também copiado a `shared/contracts/`) | Markdown alinhado ao contrato global v1.1.0 |
| Schemas das tabelas wallet (esqueleto) | `shared/schemas/wallet-tables.md` | Markdown com tabelas |
| Schema da outbox (referência) | `shared/schemas/route-event.md` | Markdown |
| Constantes | `shared/constants/payment-rules.md` | Markdown |
| ADRs iniciais | `shared/decisions/DECISIONS.md` | Markdown ADR simplificado |
| Glossário | `shared/domain/wallet-glossary.md` | Markdown |

---

### Stage 2 — Wallet core + ingestão (com publisher hook cross-projeto)

**Inputs**

| Fonte | Arquivo | Escopo | Razão |
|-------|---------|--------|-------|
| Stage 1 | `01-discovery/output/domain-model.md` | Arquivo completo | Mecânica do ledger double-entry e da conversão |
| Stage 1 | `01-discovery/output/outbox-schema.sql` | DDL da `route_events` | Migration no dispatch-engine |
| Stage 1 | `01-discovery/output/contract-extension.md` | §9, §10, §11 | Comportamento esperado do consumer |
| Stage 1 | `shared/schemas/wallet-tables.md` | Esqueleto das tabelas wallet | Migration no wallet |
| Stage 1 | `shared/constants/payment-rules.md` | Constantes | Configurar `polling_interval_seconds`, `outbox_ttl_days`, `conversion_bonus_multiplier` |
| Externo | `D:\pntrsoft\dispatch-engine\src\routers\routes.py` | Linhas 78 e 123 | Pontos onde adicionar chamada ao publisher |
| Externo | `D:\pntrsoft\dispatch-engine\src\database.py` | AsyncSession setup | Reusar pattern de transação |

**Processo**

1. **No wallet:** criar `pyproject.toml` espelhando o do dispatch-engine; criar `src/`, `tests/`, `alembic/`. Configurar ruff, pytest, alembic.
2. **No wallet:** gerar models SQLAlchemy 2.0 a partir do `wallet-tables.md` (Pydantic v2 schemas separados). Gerar migration Alembic inicial.
3. **No wallet:** implementar `src/services/credit_engine.py` que aplica regra flat (`valor_por_entrega` constante por enquanto — vem de tabela `rule` com versão ativa única no MVP).
4. **No wallet:** implementar `src/services/event_mapper.py` que mapeia `route_events.payload` em entradas double-entry no `wallet_ledger`. Suporta tipos: `route_completion` (crédito), `voucher_redeem` (débito + crédito em conta de bonificação), `breakage` (débito + crédito em receita).
5. **No wallet:** implementar `src/services/outbox_consumer.py` (poller 5s). Lê `route_events WHERE consumed_at IS NULL ORDER BY created_at LIMIT N`, processa cada um em transação, marca `consumed_at`. Idempotência via `processed_events.event_id UNIQUE`.
6. **No wallet:** implementar `_scripts/outbox-cleanup.sh` (job noturno: `DELETE FROM route_events WHERE consumed_at < NOW() - INTERVAL '90 days'`).
7. **No dispatch-engine** (cross-projeto): aplicar migration `add_route_events_outbox` (DDL do Stage 1).
8. **No dispatch-engine:** criar `src/services/route_events_publisher.py` com `publish_route_completed(route, db)` que insere row em `route_events` recebendo a mesma `db` da transação que muda `route.status`.
9. **No dispatch-engine:** chamar `publish_route_completed` em `routers/routes.py:78` (manual via PATCH) e `:123` (auto-complete). Garantir que insert outbox + update status estão na mesma transação.
10. **Cross-projeto:** escrever `tests/e2e/test_route_to_ledger.py` no wallet usando docker-compose para subir os dois serviços. Cenários: rota → completed → outbox → consumer → ledger; replay do mesmo `event_id` é idempotente; transação atômica (rollback do update do route faz rollback do insert da outbox).
11. **Observabilidade:** instrumentar `wallet_consumer_lag_seconds`, `wallet_credit_failures_total`, `wallet_ledger_balance_check_passed` (job noturno: `SUM(amount WHERE direction='credit') == SUM(amount WHERE direction='debit')` por driver).
12. **CI/CD:** habilitar `.github/workflows/verify.yml` (lint + pytest + cobertura ≥80%) no wallet. E2E em job separado com docker-compose.
13. **[Checkpoint]** — Antes de habilitar o poller em prod e antes de mergear o publisher no dispatch-engine.
14. Audit. `/cross-check` obrigatório dado o patch cross-projeto.

**Checkpoints**

| Após Passo | Agente Apresenta | Humano Decide |
|------------|-------------------|---------------|
| 12 | Resumo do PR cross-projeto: arquivos modificados no dispatch-engine + arquivos criados no wallet + cobertura de testes + métricas instrumentadas | Aprovar merge no dispatch-engine e habilitar o poller em prod |

**Verify**

```bash
# _scripts/verify-stage-2.sh
ruff check src/
ruff format --check src/
alembic upgrade head --sql > /tmp/wallet-migration-dry-run.sql
pytest tests/ -v --cov=src --cov-fail-under=80
# Idempotência:
pytest tests/integration/test_idempotency.py -v
# Double-entry balance check:
pytest tests/integration/test_ledger_balance.py -v
# E2E cross-projeto:
docker compose -f tests/e2e/docker-compose.yml up -d
pytest tests/e2e/test_route_to_ledger.py -v
docker compose -f tests/e2e/docker-compose.yml down
```

**Audit**

| Verificação | Pass condition |
|-------------|----------------|
| Ledger imutável | Sem UPDATE/DELETE em `wallet_ledger` em qualquer service. Migration do `wallet_ledger` inclui trigger ou check para prevenir mutação. |
| FK lógica entre schemas | `driver_id` UUID em `wallet_ledger` sem FK física para schema do dispatch (cross-schema FK não permitido em Supabase) |
| Transação atômica no publisher | Insert na `route_events` e UPDATE em `routes.status` na mesma `AsyncSession.begin()` no dispatch-engine |
| Logging estruturado | Toda linha de log do consumer e do publisher tem `event_id` e `route_id` em JSON |
| Sem secrets em código | `git log -p` não mostra nada com pattern de secret; `_secrets/` no `.gitignore` |
| Resiliência do poller | Falha de DB ou de mapping não derruba o processo: back-off exponencial 1→2→4→8→16→32s com jitter; reset após sucesso |
| Cobertura ≥80% | `pytest --cov-fail-under=80` passa |
| `/cross-check` passou | Skill rodou e validou os arquivos contract-sensitive antes do PR |

**Outputs**

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Models e schemas do wallet | `wallet/src/models/`, `wallet/src/schemas/` | Python |
| Migration inicial do wallet | `wallet/alembic/versions/0001_initial.py` | Python (Alembic) |
| Services do wallet | `wallet/src/services/{credit_engine,event_mapper,outbox_consumer}.py` | Python |
| Job de cleanup | `wallet/_scripts/outbox-cleanup.sh` | Bash |
| Publisher no dispatch-engine | `dispatch-engine/src/services/route_events_publisher.py` (novo) | Python |
| Migration da outbox no dispatch-engine | `dispatch-engine/alembic/versions/NNNN_add_route_events_outbox.py` | Python |
| Patches em `routers/routes.py:78` e `:123` | dispatch-engine | diff |
| Testes E2E cross-projeto | `wallet/tests/e2e/test_route_to_ledger.py` | Python + docker-compose |
| `verify.yml` GitHub Actions | `wallet/.github/workflows/verify.yml` | YAML |
| Versão final v1.1.0 do contrato global | `D:\pntrsoft\integration-contract\CONTRACT.md` | Markdown |

---

### Stage 3 — API + auth

**Inputs**

| Fonte | Arquivo | Escopo | Razão |
|-------|---------|--------|-------|
| Stage 2 | `wallet/src/models/`, `wallet/src/schemas/` | Models | Base dos endpoints |
| Stage 2 | `wallet/src/services/credit_engine.py` | Service | Cálculo de saldo |
| Stage 1 | `shared/schemas/api-spec.yaml` (esqueleto) | OpenAPI | Especificação a fechar |
| Stage 1 | `shared/contracts/integration-contract-extension.md` | §9, §10, §11 | Endpoints expostos pelo wallet |
| Externo | `D:\pntrsoft\dispatch-engine\src\routers\auth.py` | Endpoint `POST /driver/auth` | Cliente que valida `auth_code` |

**Processo**

1. Definir `openapi.yaml` v1.0.0 do wallet: endpoints `/auth/login`, `/auth/refresh`, `/me/balance`, `/me/history`, `/me/payments`, `/catalog`, `/redeem`. Request/response/errors completos.
2. Implementar `src/services/dispatch_client.py` com `validate_auth_code(auth_code: str) -> DriverSnapshot | None` chamando `POST /driver/auth` no dispatch-engine via httpx async (timeout 5s, sem retry — falha rápido).
3. Implementar `src/routers/auth.py`: `POST /auth/login` (auth_code → JWT access + refresh) e `POST /auth/refresh`. JWT HS256 com secret rotacionável; access TTL 30min; refresh TTL 24h. Rate limit por IP+auth_code (5 tentativas/min).
4. Implementar `src/middlewares/jwt.py` para verificar JWT em rotas protegidas.
5. Implementar `src/routers/wallet.py`: `GET /me/balance` (usa view materializada se disponível, senão soma do ledger), `GET /me/history` (paginado), `GET /me/payments` (semanas pagas).
6. Implementar `src/routers/marketplace.py`: `GET /catalog` (vouchers ativos), `POST /redeem` (transação: gera `voucher_redemption` + entrada no `wallet_ledger`).
7. Log scrubbing: middleware que remove campos `auth_code`, `password`, `token` antes de logar.
8. Observabilidade: instrumentar `wallet_auth_attempts_total{status}`, `wallet_auth_latency_seconds`, `wallet_dispatch_validation_failures_total`, `wallet_redemption_attempts_total{result}`.
9. Contract tests (pytest) para auth flow: login válido, login inválido, JWT expirado, refresh, rate limit.
10. Atualizar `shared/schemas/api-spec.yaml` para versão final.
11. **[Checkpoint]** — Revisar fluxo de auth e escopo do JWT (claims, expiração, secret management).
12. Audit.

**Checkpoints**

| Após Passo | Agente Apresenta | Humano Decide |
|------------|-------------------|---------------|
| 11 | OpenAPI completo + suite de contract tests + relatório de log scrubbing + plano de rotação do secret JWT | Aprovar segurança do auth e shape final da API |

**Verify**

```bash
# _scripts/verify-stage-3.sh
ruff check src/
ruff format --check src/
pytest tests/ -v --cov=src --cov-fail-under=80
spectral lint shared/schemas/api-spec.yaml
pytest tests/contract/test_auth_flow.py -v
pytest tests/security/test_log_scrubbing.py -v
pytest tests/security/test_rate_limit.py -v
```

**Audit**

| Verificação | Pass condition |
|-------------|----------------|
| JWT TTL curto | Access ≤30min, refresh ≤24h |
| Algoritmo declarado | HS256 ou RS256 documentado em `shared/decisions/DECISIONS.md` |
| Sem auth_code em logs | Teste `test_log_scrubbing.py` cobre 100% dos paths |
| CORS restrito | Lista explícita de origens (não `*`) |
| Rate limit | Por IP+auth_code, 5 tentativas/min, retorna 429 |
| Secret rotation | Documentado e testado (rotação não invalida sessões existentes durante grace period) |
| `dispatch_client` falha rápido | Timeout 5s, sem retry; falha gera 503 no `/auth/login` |

**Outputs**

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Routers | `wallet/src/routers/{auth,wallet,marketplace}.py` | Python FastAPI |
| Cliente do dispatch-engine | `wallet/src/services/dispatch_client.py` | Python httpx |
| Middleware JWT | `wallet/src/middlewares/jwt.py` | Python |
| OpenAPI final | `shared/schemas/api-spec.yaml` | YAML |
| Contract tests | `wallet/tests/contract/` | Python pytest |

---

### Stage 4 — Painel manager (Jinja2 + sessão, Auto + safety-by-design)

**Inputs**

| Fonte | Arquivo | Escopo | Razão |
|-------|---------|--------|-------|
| Stage 3 | `shared/schemas/api-spec.yaml` | OpenAPI | Endpoints internos consumidos pelo manager |
| Stage 2 | Models e schemas do wallet | Tabelas `voucher`, `rule`, `manager_audit`, `manager_user` | CRUD do manager |
| Stage 1 | `shared/constants/payment-rules.md` | Constantes | Defaults nos formulários |
| Workflow-map | §5 Stage 4 (deliverables safety-by-design) | Modal de preview matemático + audit log com diff | Requisito obrigatório |

**Processo**

1. Implementar `src/services/manager_auth.py` (login server-side com `itsdangerous` + cookie `httpOnly + Secure + SameSite=Strict`; password hashing via `argon2-cffi`).
2. Implementar `src/middlewares/csrf.py` (token em todos os POSTs).
3. Implementar `src/routers/admin.py` com endpoints Jinja2: `/admin/login`, `/admin/dashboard`, `/admin/rules`, `/admin/catalog`, `/admin/payments`, `/admin/audit`.
4. Templates `src/templates/`: base.html (layout), login.html, rules.html (CRUD com modal de preview), catalog.html (CRUD voucher com modal de preview), dashboard.html (saldos por entregador), payments.html (preview da quinta + cancelar com modal), audit.html (visualizar diffs).
5. **Modal de preview matemático em `valor_por_entrega`**: ao salvar, JS calcula impacto sobre últimos 7 dias via fetch a endpoint interno `GET /admin/api/rules/preview?new_value=X`. Backend retorna `{deliveries_last_7_days: 87, current_value: 0.50, new_value: X, weekly_delta: ..., warning: ...}`. Sem confirmação explícita no modal, formulário não submete.
6. **Mesma mecânica para criar/editar voucher** (preview de quantidade ativa e de saldo médio dos motoboys vs preço).
7. **Mesma mecânica para cancelar pagamento da quinta** (modal mostra valor total e número de motoboys afetados).
8. **Audit log**: cada `POST` que muda `rule`, `voucher`, ou `payment_cycle` insere row em `manager_audit` com `actor_id`, `action`, `entity`, `entity_id`, `before_snapshot_json`, `after_snapshot_json`. Tela `/admin/audit` permite ver diffs.
9. Bootstrap: migration Alembic em `0002_seed_initial_manager.py` cria primeiro manager usando `{{MANAGER_INITIAL_USER_EMAIL}}` e `{{MANAGER_INITIAL_USER_PASSWORD_HASH}}` (não vai ter CLI).
10. RBAC mínimo: enum `role IN ('manager', 'admin')` no `manager_user`. `admin` pode criar outros managers; `manager` só usa.
11. Smoke E2E com Playwright: `tests/e2e/test_manager_panel.py` cobre login, criar regra com preview, editar voucher, ver audit log diff, cancelar pagamento.
12. Observabilidade: `manager_actions_total{action,entity}`, `manager_login_failures_total`, alerta no log quando `valor_por_entrega` muda ou pagamento é cancelado.
13. Audit.

**Checkpoints** (não obrigatório, sugerido)

| Após Passo | Agente Apresenta | Humano Decide |
|------------|-------------------|---------------|
| 11 (opcional) | UX do modal de preview, conteúdo do audit log diff | Validar UX da safety-by-design antes do merge final |

**Verify**

```bash
# _scripts/verify-stage-4.sh
ruff check src/
ruff format --check src/
pytest tests/ -v --cov=src --cov-fail-under=80
# CSRF, modal, audit log obrigatórios:
pytest tests/integration/test_csrf.py -v
pytest tests/integration/test_preview_modal.py -v       # não persiste sem confirmação
pytest tests/integration/test_audit_log_diff.py -v       # toda mudança gera row
playwright test tests/e2e/test_manager_panel.py
```

**Audit**

| Verificação | Pass condition |
|-------------|----------------|
| Cookie seguro | `httpOnly + Secure + SameSite=Strict` em todos os ambientes (dev pode usar `Secure=False` mas warn) |
| Password hashing | `argon2id` com parâmetros recomendados (m=64MB, t=3, p=4) |
| CSRF | Token em 100% dos POSTs; teste cobre CSRF inválido → 403 |
| RBAC | `admin`-only endpoints retornam 403 para `manager`; teste cobre |
| Audit log | Toda mudança em `rule`, `voucher`, `payment_cycle` gera row em `manager_audit` (assertable: `count(audit) == count(mutations)` em teste fixture) |
| Modal de preview | `valor_por_entrega` change sem confirmação → assertion fails (teste forçado) |
| Bootstrap manager | Migration `0002_seed_initial_manager.py` cria UM manager ativo com role `admin`; idempotente (UPSERT em vez de INSERT) |

**Outputs**

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Routers do admin | `wallet/src/routers/admin.py` | Python FastAPI |
| Auth do manager | `wallet/src/services/manager_auth.py` | Python |
| Middleware CSRF | `wallet/src/middlewares/csrf.py` | Python |
| Templates | `wallet/src/templates/*.html` | Jinja2 |
| Static (CSS + JS vanilla) | `wallet/src/static/css/*.css`, `wallet/src/static/js/*.js` | CSS, JS |
| Bootstrap migration | `wallet/alembic/versions/0002_seed_initial_manager.py` | Python (Alembic) |
| E2E manager | `wallet/tests/e2e/test_manager_panel.py` | Python Playwright |

---

### Stage 5 — PWA entregador + marketplace

**Inputs**

| Fonte | Arquivo | Escopo | Razão |
|-------|---------|--------|-------|
| Stage 3 | `shared/schemas/api-spec.yaml` | OpenAPI | Cliente API |
| Stage 4 | Catálogo de vouchers ativo (via API) | Dados runtime | PWA consome o catálogo publicado pelo manager |
| Stage 1 | `shared/domain/wallet-glossary.md` | Glossário | Terminologia consistente nas telas (saldo, conversão, voucher, breakage) |
| Stage 1 | `shared/constants/payment-rules.md` | Constantes | Mostrar dia de pagamento, multiplicador de bônus, prazos de expiração |

**Processo**

1. Inicializar `pwa/` com Vite + React 18 + TS strict + ESLint + vitest. `package.json` com scripts `dev`, `build`, `lint`, `typecheck`, `test`.
2. Configurar `manifest.json` (PWA: name, icons, theme_color), service worker para cache offline básico do shell + último saldo conhecido.
3. Implementar `pwa/src/services/api.ts` (cliente fetch tipado a partir do `api-spec.yaml`).
4. Implementar `pwa/src/services/fcm.ts` (cliente FCM próprio com credenciais separadas via env `VITE_FCM_API_KEY`).
5. Implementar pages: `LoginPage` (auth_code), `DashboardPage` (saldo + entregas hoje + dia de pagamento), `HistoryPage` (paginado), `PaymentPage` (saldo a pagar quinta = creditado − convertido), `MarketplacePage` (catálogo), `RedeemPage` (modal de confirmação tudo-ou-nada), `VouchersPage` (resgatados ativos com QR/código), `ConvertPage` (conversão com preview do bônus 1.10x).
6. **Aviso de expiração**: rota interna que checa `expiry_alert` e mostra banner; FCM push agendado 30 e 7 dias antes.
7. Tratamento de auth: armazena `access_token` em `localStorage` + `refresh_token` em cookie httpOnly (via API `/auth/refresh`). Renova access automaticamente.
8. Observabilidade frontend: enviar para endpoint do backend `wallet_pwa_fcm_send_failures_total`, `wallet_pwa_redemption_rate`, `wallet_pwa_balance_view_latency_seconds`. Logging via `console.error` + envio batch ao backend.
9. CI/CD: habilitar `.github/workflows/build-pwa.yml` (eslint + tsc + vitest + vite build). Habilitar `deploy.yml` (Railway PWA + alembic upgrade head em prod).
10. Playwright E2E: fluxo crítico (login → ver saldo → converter → resgatar voucher → ver código).
11. **[Checkpoint]** — Revisar telas críticas (conversão, checkout, expiração).
12. Audit.

**Checkpoints**

| Após Passo | Agente Apresenta | Humano Decide |
|------------|-------------------|---------------|
| 11 | Tela de conversão (com preview do bônus), tela de checkout (tudo-ou-nada), banner de expiração 30/7 dias | Aprovar UX antes de deploy em prod |

**Verify**

```bash
# _scripts/verify-stage-5.sh
cd pwa/
npm run lint
npm run typecheck
npm test -- --coverage
npm run build
playwright test tests/e2e/test_pwa_critical.spec.ts
# deploy dry-run:
railway up --dry-run --service pwa
```

**Audit**

| Verificação | Pass condition |
|-------------|----------------|
| Offline-friendly | Service worker cacheia shell + último saldo conhecido; abre sem rede e mostra saldo cacheado com badge "última atualização: X" |
| Token storage | Access em `localStorage` (XSS protegido por React + CSP); refresh em cookie httpOnly |
| Voucher único | UUID + assinatura HMAC; backend rejeita re-resgate da mesma row |
| FCM token rotation | Renova periodicamente e envia ao backend; backend desconsidera tokens antigos |
| Push permission flow | Pede permissão depois do primeiro login e exibe explicação clara |
| TS strict | `tsc --noEmit` sem warnings; `strict: true` no `tsconfig.json` |
| Build PWA | `vite build` produz `dist/` ≤500KB gzipped |

**Outputs**

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Pages | `wallet/pwa/src/pages/*.tsx` | TypeScript React |
| Services | `wallet/pwa/src/services/{api,fcm,storage}.ts` | TypeScript |
| Manifest e SW | `wallet/pwa/public/manifest.json`, `wallet/pwa/src/service-worker.ts` | JSON, TS |
| Build PWA | `wallet/pwa/dist/` (artefato CI) | estático |
| `build-pwa.yml`, `deploy.yml` | `wallet/.github/workflows/` | YAML |
| E2E PWA | `wallet/pwa/tests/e2e/test_pwa_critical.spec.ts` | Playwright |

---

### Stage 6 — Engine de promoções avançadas (opcional)

**Inputs**

| Fonte | Arquivo | Escopo | Razão |
|-------|---------|--------|-------|
| Stages 1–5 | Output completo | Sistema estável em produção | Pré-requisito do escopo opcional |
| Stage 4 | `manager_audit` schema, modal de preview | Pattern de safety | Reusar safety-by-design no painel de regras compostas |
| Stage 2 | `wallet_ledger`, `rule`, `rule_application` schemas | Tabelas | Persistir versões de regras |

**Processo**

1. Implementar `src/services/rule_engine.py` com predicados ordenados (lista de regras com `priority`, `condition` em DSL pequena tipo `event.delivery_count_today >= 15`, `multiplier`, `valid_from`, `valid_until`). Primeira regra que dá match aplica.
2. Estender `src/models/rule.py`: campos `version` (auto-increment por nome de regra), `condition_dsl` (string), `multiplier` (decimal), `valid_from`, `valid_until`, `created_by` (manager).
3. Modificar `credit_engine.py` para chamar `rule_engine.apply(event)` em vez de aplicar `valor_por_entrega` direto. Manter compatibilidade com regra flat (regra default `multiplier=1.0`).
4. Persistir `rule_application`: qual `rule.id`+`version` aplicou em qual entrada do ledger (`event_id`).
5. Implementar painel manager para regras compostas (Jinja2). Reusar modal de preview matemático (Stage 4).
6. Implementar **simulador dry-run**: aplica novo conjunto de regras sobre últimos 30 dias do `wallet_ledger` (snapshot, sem persistir) e retorna diff total e por motoboy.
7. Testes de regressão: regra default flat continua creditando idêntico aos últimos 30 dias.
8. Observabilidade: `wallet_rule_applications_total{rule_id,version}`, `wallet_rule_simulator_runs_total`, alerta no log quando regra nova é ativada (audit log com diff).
9. Audit.

**Checkpoints** — não obrigatório.

**Verify**

```bash
# _scripts/verify-stage-6.sh
ruff check src/
ruff format --check src/
pytest tests/ -v --cov=src --cov-fail-under=80
# Regressão: regras flat continuam funcionando idênticas
pytest tests/regression/test_flat_rule_unchanged.py -v
# Simulador dry-run sobre snapshot
pytest tests/simulation/test_rule_simulator.py -v
```

**Audit**

| Verificação | Pass condition |
|-------------|----------------|
| Regras imutáveis | Tabela `rule` versionada (UPDATE proibido); nova versão = nova row com `version+1` |
| Audit trail | `rule_application` registra `rule.id+version` para cada entrada do ledger |
| Simulador dry-run | Não persiste; resultado em memory/response apenas |
| Regressão flat | Regra default continua produzindo identicamente aos últimos 30 dias do ledger |
| Painel reusa safety | Modal de preview obrigatório ao criar/editar regra (mesma mecânica do Stage 4) |

**Outputs**

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Rule engine | `wallet/src/services/rule_engine.py` | Python |
| Models de regras | `wallet/src/models/rule.py` (estendido) | Python SQLAlchemy |
| Migration | `wallet/alembic/versions/NNNN_extend_rules.py` | Python |
| Painel manager regras compostas | `wallet/src/templates/rules_composite.html` + `wallet/src/routers/admin.py` extension | Jinja2 + Python |
| Simulador | `wallet/src/services/rule_simulator.py` | Python |
| Testes de regressão | `wallet/tests/regression/`, `wallet/tests/simulation/` | Python pytest |

---

## 5. Verificação do Grafo

### 5.1 Sem referências circulares

DAG inspecionado:

- 1 → 2 (contracts/schemas)
- 2 → 3 (models/services)
- 3 → 4 (mesma app FastAPI; imports Python, **não HTTP**)
- 3 → 5 (API HTTP cross-process)
- 4 → 6 (UI de regras compostas + simulador no manager; reusa Jinja2 e safety-by-design)
- 2 → 6 (modifica `credit_engine.py` e usa schema `wallet_ledger`)

Arestas que **não** existem (validar a ausência delas é tão importante quanto declarar as existentes):

- ~~4 → 5~~: catálogo é consultado via API HTTP em runtime, não é build-time dependency.
- ~~5 → 6~~: PWA mostra "R$ X recebido" indiferente se flat ou composto; sem mudança.
- ~~6 → 5~~: o oposto também não existe.

Nenhum stage downstream alimenta upstream. Confirmação: unidirecional.

### 5.2 Todo output consumido

| Stage | Output | Consumido por |
|-------|--------|---------------|
| 1 | `domain-model.md`, `contract-extension.md`, `outbox-schema.sql`, `wallet-tables.md`, `payment-rules.md`, `wallet-glossary.md`, `DECISIONS.md` | 2 (todos), 3, 4, 5, 6 (subset por stage) |
| 2 | models, schemas, services (incl. publisher cross-projeto), CI verify.yml, `route_events` em prod | 3 (mesma app), 4 (mesma app), 5 (API consome models via OpenAPI), 6 (extensão do `credit_engine`) |
| 3 | `api-spec.yaml`, routers, middleware JWT | 4 (mesma app FastAPI — imports diretos), 5 (consumir API HTTP), 6 (consultar dados via mesma app) |
| 4 | Painel + audit log + bootstrap manager + Jinja2 base | 6 (painel de regras compostas reusa Jinja2 e safety-by-design) |
| 5 | PWA + deploy.yml | (terminal — produto final para entregador) |
| 6 | Rule engine + simulador + painel composto | (terminal — opcional) |

Nenhum output órfão. Outputs terminais (5 e 6) são entregáveis ao usuário final.

### 5.3 Fontes canônicas verificadas

Conferido em §3. Cada fato tem uma única casa. Nenhuma duplicação de definição.

---

## 6. Stages Paralelizáveis

| Par | Paralelizável? | Justificativa |
|-----|----------------|---------------|
| 1 ↔ 2 | Não | 2 lê output de 1 |
| 2 ↔ 3 | Não | 3 lê models de 2 |
| 3 ↔ 4 | Não | 4 estende mesma app FastAPI de 3 |
| 3 ↔ 5 | Não | 5 lê `api-spec.yaml` de 3 |
| **4 ↔ 5** | **Sim** | Após 3 completo, 4 e 5 não dependem entre si — 5 consulta o catálogo via API HTTP em runtime (não é build-time dependency). Podem rodar em duas sessões/devs paralelos. |
| **5 ↔ 6** | **Sim** | 6 modifica backend (`credit_engine.py`) e UI do manager (Jinja2). PWA não muda. Acadêmico para solo dev mas o DAG não mente. |
| 4 ↔ 6 | Não | 6 estende o manager de 4 |
| 2 ↔ 6 | Não | 6 modifica `credit_engine.py` de 2 |

Recomendação prática: solo dev pode fazer Stage 4 primeiro (Auto + safety) e depois Stage 5 (Review-gate, mais demorado). Ou inverter se quiser PWA antes do manager (válido — manager pode ser populado via `psql` direto no MVP).

---

## 7. Validação dos Audits do Builder Stage 03

| Verificação | Status |
|-------------|--------|
| Sem referências circulares | ✅ §5.1 |
| Consumo de output | ✅ §5.2 |
| Completude de contrato | ✅ §4 (todos os 6 stages têm Inputs/Processo/Verify/Audit/Outputs) |
| Fontes canônicas | ✅ §3 (zero duplicação) |
| Seções Verify | ✅ §4 (todos os stages que produzem código têm bloco Verify executável) |
| Tipos de dependência | ✅ §2 (file/package/schema separados) + §2.4 cross-projeto explícito |

---

## 8. Pendências para o Stage 04 (Scaffolding) do builder

Stage 04 vai gerar a árvore de arquivos do workspace `wallet-build` em `D:\pntrsoft\workspaces\wallet-build\` baseado neste documento. Antes:

1. **Templates esperados** (Stage 04 do builder usa `references/templates/`, todos confirmados existentes):
   - `workspace-claude-template.md` → CLAUDE.md raiz do `wallet-build` (placeholders: `{{NOME_PROJETO}}`, `{{DESCRICAO_CURTA}}`, `{{FOLDER_MAP}}`, `{{TRIGGERS_TABLE}}`, `{{ROUTING_TABLE}}`, `{{LOADING_TABLE}}`)
   - `workspace-context-template.md` → CONTEXT.md raiz do `wallet-build`
   - `progress-template.md` → PROGRESS.md raiz do `wallet-build` (não do codebase `wallet`)
   - `stage-context-template.md` → CONTEXT.md de cada um dos 6 stages do `wallet-build` (placeholders: `{{STAGE_NUMBER}}`, `{{STAGE_NAME}}`, etc., com seções condicionais `{{?CHECKPOINTS}}`, `{{?VERIFY}}`, `{{?AUDIT}}`)
   - `claude-md-dev.md` → CLAUDE.md do **codebase** `wallet/` (depois do scaffold do produto, em `D:\pntrsoft\wallet\CLAUDE.md`) — NÃO o do workspace
   - `ci-workflow-template.md` → ver item 9 abaixo
2. **Convenção de nome do workspace**: `wallet-build` — segue padrão `<escopo>-<ação>` dos 6 workspaces existentes. `wallet` (cru) reservado para o codebase do produto em `D:\pntrsoft\wallet\`.
3. **Placeholders a injetar nos templates** (do §7 do workflow-map): 16 no total. Coletados via `setup/questionnaire.md` do `wallet-build`.
4. **Skills a copiar (Pattern 4)**: `/cross-check` (com lista própria do `wallet-build`, incluindo cross-projeto), `/test`, `/migrate`. Verificar convenção do dispatch-engine (provavelmente `.claude/agents/` ou `.claude/commands/`).
5. **Diretórios obrigatórios a criar dentro de `wallet-build/`**:
   - `setup/`, `stages/{01-discovery,02-core,03-api,04-manager,05-pwa,06-promotions}/{output,references}/`, `shared/{domain,schemas,contracts,decisions,constants,references}/`, `_scripts/`, `.github/workflows/` (este último para os workflows que **constroem** o wallet-build, não para o codebase wallet)
6. **Arquivos boilerplate a deixar prontos em `wallet-build/`**: `PROGRESS.md`, `DECISIONS.md` (no `shared/decisions/`), `CHANGELOG.md`, `README.md`, `CLAUDE.md`, `CONTEXT.md`, `.gitignore`, `setup/questionnaire.md`. **Não criar `pyproject.toml` ou `pwa/package.json` no wallet-build** — esses pertencem ao codebase `wallet/` e nascem no Stage 02 do wallet-build.
7. **Arquivos `output/` e `references/` por stage**: criar pastas vazias (com `.gitkeep`); referências do stage podem ser populadas no Stage 04 ou deixadas vazias para o usuário preencher conforme avança.

### Itens novos (8, 9, 10) — fechar antes do Scaffolding

8. **Conteúdo inicial dos arquivos boilerplate** (não inventar — usar templates):
   - `wallet-build/CLAUDE.md` ← `workspace-claude-template.md` com placeholders preenchidos: `{{NOME_PROJETO}}=wallet-build`, `{{DESCRICAO_CURTA}}=Workspace ICM para construir o sistema wallet (stored value para entregadores)`, `{{FOLDER_MAP}}` derivado de §5 deste documento, `{{TRIGGERS_TABLE}}=setup, status, cross-check`, `{{ROUTING_TABLE}}` linkando os 6 stages, `{{LOADING_TABLE}}` derivado da matriz de dependências em §2.
   - `wallet-build/CONTEXT.md` ← `workspace-context-template.md` similar.
   - `wallet-build/PROGRESS.md` ← `progress-template.md` com estado inicial **não vazio**: `{{CURRENT_PHASE}}=Stage 01 (Discovery & contract) — pending`, sessão inicial = "Workspace gerado pelo workspace-builder-v2 em 2026-04-25", próximos passos = lista derivada do Processo do Stage 01, e os 6 stages listados como pendentes.
   - `wallet-build/shared/decisions/DECISIONS.md` ← **não vazio**: pre-populado com as **17 decisões** já tomadas até agora (G1, G2, G3, G4, G7, Q5, Q6, Q7, Q12, Q14, Q3+Q9, Q8, Q2-residual, Q13 + as 3 working assumptions Q11/Q10/Q13 com status `pending-product-validation`/`blocked-by-external-validation`/`with-review-checkpoint`). Stage 1 só **complementa**, não cria do zero.
   - `wallet-build/CLAUDE.md` cada stage: `stage-context-template.md` preenchido com Inputs/Processo/Verify/Audit/Outputs canônicos do §4 deste documento. **Cada um dos 6 stages tem seu CONTEXT.md já materializado** — o usuário não precisa preencher.
   - `wallet-build/.gitignore`: padrão Python + Node + `_secrets/` + `.env` + `_scripts/*.local`.

9. **Localização dos scripts de Verify** (Pattern 11): **`wallet-build/_scripts/`** na raiz do workspace, **não dentro de cada stage**. Pattern alinhado com `fullstack-api-summary.md` do builder. Nomes:
   - `wallet-build/_scripts/verify-stage-1.sh` ... `verify-stage-6.sh`
   - `wallet-build/_scripts/run-all-checks.sh` (executa 1–5 em sequência; 6 sob demanda)
   - `wallet-build/_scripts/outbox-cleanup.sh` (job noturno mencionado em Stage 2)
   
   **Nota**: estes scripts operam sobre o codebase `wallet/` (que ainda nem existe no Stage 1). Stage 04 cria os arquivos com placeholders que serão atualizados pelo Stage 02 do wallet-build (quando o `wallet/` nascer). `ci-workflow-template.md` do builder gera `wallet/.github/workflows/verify.yml` (no codebase, não no workspace) — Stage 02 do wallet-build faz isso.

10. **Convenção de skill local cross-projeto**: `/cross-check` no `wallet-build` precisa ler **dois manifestos**: o local (arquivos do `wallet/`) e o cross-projeto (arquivos do `D:\pntrsoft\dispatch-engine\`). Stage 04 cria o template do skill em `wallet-build/.claude/commands/cross-check.md` com lista declarativa das duas categorias. Lista preenchida pelo Stage 1 do wallet-build (que finaliza a extensão do contrato e identifica os arquivos contract-sensitive específicos).

### Defaults a registrar no `DECISIONS.md` no momento do Scaffolding

Stage 04 do builder pre-popula o `DECISIONS.md` com:

- **JWT HS256 (permanente)**: wallet é one-issuer/one-verifier (mesma app FastAPI). HS256 com secret simétrico em `.env`. Status: `active-permanent`. Não migrar para RS256 sem mudança de arquitetura (multi-issuer ou separação de trust boundary).
- **Observabilidade (working default)**: `structlog` + stdout (Railway captura via tail de logs). Sem métricas dedicadas no MVP. Status: `working-default-revisit-at-go-live`. Trigger de revisão: 1º incidente operacional ou volume justificar (não preventivo). Quando revisitar, candidatos: Prometheus client, OpenTelemetry, log-based metrics em Grafana Cloud.
- As 17 decisões mencionadas no item 8.
