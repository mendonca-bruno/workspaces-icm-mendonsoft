# Stage 02 — Mapa de Workflow: Workspace `wallet`

**Data:** 2026-04-25
**Workspace alvo:** `wallet` (a ser criado em `D:\pntrsoft\workspaces\wallet\`)
**Domínio:** stored value ecosystem para entregadores da hamburgueria
**Idioma do workspace:** português (alinhado com 4/6 dos workspaces ICM existentes do monorepo)
**Inputs deste stage:** `../01-analysis/output/analysis-report.md` + respostas do Discovery

---

## 1. Resumo do Domínio

Sistema interno de stored value para motoboys já cadastrados no `dispatch-engine`. Quando uma rota muda para `completed`, o `dispatch-engine` registra o evento numa **outbox `route_events`** dentro do seu próprio schema. O `wallet` consome essa outbox via polling (5s), aplica regras de remuneração configuradas pelo manager, e credita o saldo na carteira do entregador via **ledger double-entry imutável** indexado pelo `event_id` da outbox (idempotência natural).

O entregador acompanha entregas, saldo e histórico em uma **PWA React + Vite + TS**, e pode **converter parte do saldo em vouchers** de itens da hamburgueria com **bônus de conversão de 1.10x** (modelo Starbucks misto). O que não for convertido sai em **dinheiro às quintas-feiras**, preservando o status quo de pagamento. **Vouchers são códigos validados manualmente no balcão** (sem integração POS no MVP). Saldo expira em **6 meses**, com avisos via FCM 30 e 7 dias antes; saldo expirado vira **breakage** (receita).

O manager (1–2 pessoas) usa um **painel server-side em FastAPI + Jinja2 com sessão httpOnly** (não replica o admin público do dispatch-engine), gerencia **regras de remuneração flat** no MVP (R$ X por entrega), publica vouchers no catálogo, audita saldos e o pagamento semanal.

---

## 2. Decisões Fixadas (não reabrir)

Espelho do §7 do `analysis-report.md` + decisões do Discovery:

| Tema | Decisão |
|------|---------|
| **G1 — Auth cross-system** | Login no wallet com mesmo `auth_code` 6 dígitos do dispatch-engine. Wallet emite seu **próprio JWT curto**. Dispatch-engine intocado. |
| **G2 — Evento `route.completed`** | **Outbox** `route_events` no schema do dispatch-engine. Wallet faz polling. Sem webhook HTTP. |
| **G3 — Idempotência** | `event_id` da outbox como chave única no ledger. |
| **G4 — Auth do manager** | Auth real desde dia 1. Jinja2 + `itsdangerous` + cookie httpOnly + secure + CSRF. |
| **G7 — CPF no driver** | **Destravado / fora do MVP** — Q14 escolheu modelo de voucher/bonificação sem cupom fiscal nominal. |
| **Q12 — Fungibilidade no checkout** | "Tudo ou nada" no MVP. Split fica para Stage 6+. |
| **Q5 — Marketplace MVP** | Catálogo apenas hamburgueria + vouchers + sem decremento de estoque real. |
| **Q6 — Regras de remuneração MVP** | Flat (`valor_por_entrega`). Engine de regras compostas no Stage 6 opcional. |
| **Q7 — Ciclo de pagamento** | Quinta-feira. Sem cut-off (saldo conversível a qualquer hora). Pagamento em dinheiro = saldo creditado − saldo já convertido. |
| **Q3+Q9 — Manager framework** | FastAPI + Jinja2 + sessão server-side. |
| **Q8 — FCM** | **Wallet tem seu próprio cliente FCM** com credenciais Firebase separadas. |
| **Q2-residual — Outbox polling** | 5 segundos. Marca `consumed_at`. TTL 90 dias com job noturno. Sem CDC/offset. |
| **Q14 — Resgate downstream** | Voucher validado manualmente no balcão. Sem integração POS no MVP. Sem cupom fiscal nominal — bonificação. |
| **Q13 — Expiração** | 6 meses. Aviso 30 e 7 dias antes via FCM. Breakage vira receita. |
| **Idioma e estilo** | Português, sem emojis, técnico-imperativo, tabelas markdown. |

---

## 3. Decisões Reservadas (working assumptions com flag)

Estas alimentam o workflow mas estão sinalizadas em `shared/decisions/DECISIONS.md` para revisão:

| ID | Working assumption | Status | Cascata se mudar |
|----|--------------------|--------|------------------|
| Q11 | **(c) misto** — saldo creditado, parcela conversível em voucher (bônus 1.10x), restante pago em dinheiro na quinta | `decision-pending-product-validation` | Mudança para (b) puro: catálogo vira único caminho de saída do saldo; Stage 5 perde tela "valor a pagar na quinta"; impacto regulatório (BACEN — arranjo de pagamento fechado). |
| Q10 | Motoboys autônomos/MEI; saldo = adiantamento de remuneração (passivo); conversão em voucher = bonificação (despesa) | `decision-blocked-by-external-validation` (contador) | Schema do ledger não pode fechar até contador validar. Stage 03 do builder (Mapping) registra a flag. |
| Q13 | Expiração 6 meses | `decision-with-review-checkpoint` | Revisar após 90 dias de produção. Pode subir para 12m sem refactor (config var). |

---

## 4. Fluxo de Stages

```
[1 Discovery & contract] → [2 Wallet core + ingestão] → [3 API + auth] → [4 Painel manager] → [5 PWA + marketplace]
                                                                                                          │
                                                                                                          ↓
                                                                                              [6 Promoções avançadas (opcional)]
```

DAG estritamente sequencial 1→5. Stage 6 ramifica de 5 e roda só sob demanda.

---

## 5. Stages — Contratos Formais

### Stage 1 — Discovery & contract

| Item | Valor |
|------|-------|
| **Propósito** | Modelar domínio (wallet, ledger, evento, voucher, breakage, conversão), produzir extensão do `D:\pntrsoft\integration-contract\CONTRACT.md`, definir schema da outbox `route_events`. |
| **Input** | Decisões fixadas (§2), decisões reservadas (§3), `D:\pntrsoft\integration-contract\CONTRACT.md` v1.0.0, `D:\pntrsoft\dispatch-engine\src\models\route.py` e `driver.py`, `D:\pntrsoft\dispatch-engine\.ai-context\backend.md` |
| **Output** | `output/domain-model.md` (glossário + entidades) + `output/contract-extension.md` (patch do contrato global, seções 9 wallet, 10 OutboxEvent, 11 schemas wallet, changelog v1.1.0) + `output/outbox-schema.sql` (DDL da `route_events` no schema dispatch) |
| **Verify** | markdownlint do contrato extendido; SQL DDL passa em `EXPLAIN`/dry-run |
| **Audit** | Convenção do contrato preservada (numeração, "Mudanças breaking", changelog, lista contract-sensitive); `event_id UUID PK` definido; transição `route.completed` → outbox documentada nos 2 pontos (`routes.py:78` e `:123`); idempotência declarada |
| **Automation** | **Colaborativo** (decisões de produto/contrato — usuário no loop) |
| **Checkpoint** | Sim — revisão do contrato extendido antes de mexer em código |

---

### Stage 2 — Wallet core + ingestão (com publisher hook no dispatch-engine)

| Item | Valor |
|------|-------|
| **Propósito** | Schema próprio do wallet (Postgres/Supabase), migrations, ledger double-entry imutável, polling da outbox, aplicação de regras flat, crédito idempotente. **+ publisher hook no dispatch-engine** que insere em `route_events` na transição para `completed`. Sem isso, o consumer fica sem produtor. |
| **Input** | `domain-model.md` + `contract-extension.md` + `outbox-schema.sql` do Stage 1 |
| **Output (wallet)** | `src/models/`, `src/schemas/`, `alembic/versions/`, `src/services/outbox_consumer.py` (poller 5s), `src/services/event_mapper.py` (route → ledger entries), `src/services/credit_engine.py` (aplica regra flat), tabela `wallet_ledger` (double-entry), tabela `processed_events` (event_id UNIQUE), config `polling_interval_seconds=5`, `outbox_ttl_days=90` |
| **Output (dispatch-engine — modificação cross-projeto)** | (a) `D:\pntrsoft\dispatch-engine\src\services\route_events_publisher.py` — função `publish_route_completed(route, db)` que insere row em `route_events` (mesma transação Alembic do `route.status=completed`); (b) chamadas em `D:\pntrsoft\dispatch-engine\src\routers\routes.py:78` (manual) e `:123` (auto-complete); (c) migration Alembic do dispatch-engine criando tabela `route_events`; (d) índice em `(consumed_at, created_at)` para o poller. **Mudança transacional:** insert na outbox + update do route.status na **mesma transação** — atomicidade garantida. |
| **Output (testes E2E cross-projeto)** | `tests/e2e/test_route_to_ledger.py` no wallet: sobe ambos via docker-compose, dispara `PATCH /routes/{id}/status` no dispatch, verifica row em `route_events`, espera ≤10s, verifica entrada no `wallet_ledger` com `event_id` correto. **Testa também replay** (re-insert mesmo event_id → consumer ignora). |
| **Verify** | `ruff check src/`; `ruff format --check src/`; `alembic upgrade head --sql` (dry run dos dois projetos); `pytest tests/ -v --cov=src --cov-fail-under=80`; **teste de idempotência** (replay mesmo `event_id` não credita 2x); **teste de double-entry** (sum debits == sum credits por driver); **teste E2E publisher→outbox→consumer→ledger**; **observabilidade**: emite métrica `wallet_consumer_lag_seconds` (tempo entre `route_events.created_at` e `processed_events.processed_at`), `wallet_credit_failures_total`, `wallet_ledger_balance_check_passed` (job diário). |
| **Audit** | Ledger imutável (sem UPDATE/DELETE em entries); FK lógica (`driver_id` UUID sem FK física entre schemas); insert na outbox e update do route.status na mesma transação no dispatch; logging estruturado em JSON com `event_id` em todas as linhas; sem secrets em código; falha do poller não derruba o serviço (back-off exponencial). |
| **Automation** | **Review-gate** — ledger e idempotência são críticos; publisher cross-projeto exige `/cross-check` |
| **Checkpoint** | Sim — antes de habilitar o poller em prod e antes de mergear o publisher no dispatch-engine |
| **CI/CD** | **GitHub Actions habilitado AQUI** (`.github/workflows/verify.yml` com lint+pytest no PR). E2E roda em job separado com docker-compose. |

---

### Stage 3 — API + auth

| Item | Valor |
|------|-------|
| **Propósito** | API REST do wallet: login (auth_code → JWT), saldo, histórico, extrato, conversão saldo→voucher, listagem de vouchers do entregador. |
| **Input** | Core do Stage 2 |
| **Output** | `src/routers/auth.py` (POST /auth/login: valida auth_code contra dispatch-engine, emite JWT), `src/routers/wallet.py` (GET /me/balance, /me/history, /me/payments), `src/routers/marketplace.py` (GET /catalog, POST /redeem), `src/middlewares/jwt.py`, `src/services/dispatch_client.py` (valida auth_code via httpx contra dispatch-engine), `openapi.yaml` |
| **Verify** | `ruff`, `pytest --cov-fail-under=80`, **contract tests do auth flow**, openapi lint (`spectral lint openapi.yaml`), **rate limit testado** no /auth/login, log scrubbing testado (auth_code nunca em logs); **observabilidade**: `wallet_auth_attempts_total{status}`, `wallet_auth_latency_seconds`, `wallet_dispatch_validation_failures_total`, `wallet_redemption_attempts_total{result}` |
| **Audit** | JWT com expiração curta (default 30min) + refresh token (default 24h); secret rotation documentado; algoritmo HS256 ou RS256 — escolher e documentar; sem auth_code em logs/erros; CORS restrito; rate limit por IP+auth_code no /auth |
| **Automation** | **Review-gate** — auth tem implicações de segurança |
| **Checkpoint** | Sim — fluxo de auth e escopo do JWT |

---

### Stage 4 — Painel manager (Jinja2 + sessão, **Auto + safety-by-design**)

| Item | Valor |
|------|-------|
| **Propósito** | Painel server-side para 1–2 managers: CRUD de regras de remuneração (flat MVP), CRUD do catálogo de vouchers, dashboard de saldos por entregador, log de pagamentos da quinta, auditoria do ledger. **Mais alta alavanca financeira do sistema** — typo no `valor_por_entrega` propaga para todas as entregas até alguém perceber. Por isso: Auto, mas com safety-by-design obrigatória nos pontos críticos. |
| **Input** | API do Stage 3 |
| **Output** | `src/routers/admin.py` (Jinja2 endpoints), `src/templates/` (base.html, login.html, rules.html, catalog.html, dashboard.html, payments.html, audit.html), `src/services/manager_auth.py` (login com `itsdangerous` + cookie httpOnly + secure + CSRF), `src/middlewares/csrf.py`, `src/static/css|js` (vanilla JS, nada de SPA) |
| **Output — safety-by-design (deliverables obrigatórios)** | (a) **Modal de confirmação com preview matemático** ao salvar `valor_por_entrega`: mostra valor atual, novo valor, e impacto estimado baseado em entregas dos últimos 7 dias (ex: "Última semana teve 87 entregas → impacto estimado R$ 435,00/semana"). Sem confirmação explícita, não persiste. (b) **Audit log com diff antes/depois** em toda mudança de regra de remuneração e em toda publicação/edição/remoção de voucher: tabela `manager_audit` com `actor_id`, `action`, `entity`, `before_snapshot_json`, `after_snapshot_json`, `created_at`. (c) Igual safety para cancelar/reverter pagamento da quinta — modal de confirmação obrigatório. |
| **Verify** | `ruff`, `pytest`, **smoke E2E** com Playwright nas telas críticas (login, criar regra com modal de preview, publicar voucher, ver pagamento, audit log diff), CSRF token testado, **teste obrigatório do modal de preview** (não persiste sem confirmação), **teste do audit log** (toda ação sensível gera row); **observabilidade**: `manager_actions_total{action,entity}`, `manager_login_failures_total`, eventos críticos (mudança em `valor_por_entrega`, cancelamento de pagamento) emitem alerta no log estruturado |
| **Audit** | Cookie `httpOnly + Secure + SameSite=Strict`, password hashing (`argon2` ou `bcrypt`), CSRF em todos os POSTs, RBAC mínimo (`manager` vs `admin`), audit log de ações sensíveis cobre 100% das mudanças em regras e pagamentos, modal de preview cobre 100% das mudanças em `valor_por_entrega` |
| **Automation** | **Auto + safety-by-design** — Auto cobre o ritmo do solo dev; safety-by-design cobre o risco real de typo financeiro |
| **Checkpoint** | Não obrigatório (review final no Stage 7 do builder); revisão de UX do modal de preview vale fazer antes do merge final |

---

### Stage 5 — PWA entregador + marketplace

| Item | Valor |
|------|-------|
| **Propósito** | PWA React+Vite+TS strict para o entregador: login com auth_code, saldo, histórico de entregas, dia de pagamento, conversão saldo→voucher (bônus 1.10x), checkout tudo-ou-nada de voucher, listagem de vouchers ativos, aviso de expiração via FCM. |
| **Input** | API do Stage 3 + Painel do Stage 4 |
| **Output** | `pwa/src/pages/` (LoginPage, DashboardPage, HistoryPage, MarketplacePage, VouchersPage, PaymentPage), `pwa/src/services/api.ts` (cliente JWT), `pwa/src/services/fcm.ts` (cliente FCM próprio do wallet), `pwa/manifest.json`, service worker para offline básico, telas de aviso de expiração 30/7 dias |
| **Verify** | `eslint`, `tsc --noEmit` (strict), `vitest --coverage`, `vite build`, deploy Railway dry-run, testes Playwright para fluxo crítico (login → conversão → checkout); **observabilidade**: `wallet_pwa_fcm_send_failures_total`, `wallet_pwa_redemption_rate` (vouchers resgatados / saldo creditado), `wallet_pwa_balance_view_latency_seconds`, falhas de service worker emitem evento estruturado |
| **Audit** | Offline-friendly (saldo último visto cacheado), token storage seguro (`localStorage` + escape de XSS via React + CSP), voucher único e não-replicável (UUID + assinatura no backend), FCM token rotation, push notification permission flow |
| **Automation** | **Review-gate** — UX do usuário final + checkout |
| **Checkpoint** | Sim — telas de conversão, checkout e expiração |
| **CI/CD** | **Build PWA + deploy Railway na main habilitado AQUI** |

---

### Stage 6 — Engine de promoções avançadas (opcional)

| Item | Valor |
|------|-------|
| **Propósito** | Engine de regras compostas: bônus por meta diária (ex: 15ª entrega +50%), fim de semana 1.2x, gamificação leve. **A/B testing foi cortado deste stage** — exige segmentação, métricas comparativas e hipótese registrada; vira backlog/Stage 7 hipotético se justificar. |
| **Input** | Stages 1–5 estáveis em produção |
| **Output** | `src/services/rule_engine.py` (predicados ordenados ou DSL pequena), `src/models/rule.py` (regras versionadas + período de vigência), painel de regras compostas no manager (com mesma safety-by-design de Stage 4: preview matemático ao criar/editar regra), log de aplicação de regras por evento, **simulador dry-run** que aplica novo conjunto de regras sobre últimos 30 dias e mostra diferença vs realidade. |
| **Verify** | `ruff`, `pytest`, **testes de regressão das regras flat** (não pode quebrar Stage 2), simulador de regras com snapshot de 30 dias, **observabilidade**: `wallet_rule_applications_total{rule_id,version}`, `wallet_rule_simulator_runs_total`, alerta quando regra nova é ativada |
| **Audit** | Regras versionadas e imutáveis (nova versão = nova row); audit trail de qual regra/versão creditou qual entrada do ledger; preview do simulador exibido antes de ativar regra nova |
| **Automation** | **Auto** — escopo bem definido após estabilidade do MVP |
| **Checkpoint** | Não obrigatório |
| **Excluído deste stage (backlog)** | A/B testing entre regras (segmentação por driver_id, métricas comparativas, hipótese pré-registrada). Promover a stage próprio se/quando justificar. |

---

## 6. Contexto Compartilhado (`shared/`)

Aplicando Pattern 12 do builder. Um tipo tem casa canônica; stages referenciam, não duplicam.

```
shared/
├── domain/
│   └── wallet-glossary.md          ← saldo, ledger, evento, voucher, breakage, conversão, cut-off
├── schemas/
│   ├── route-event.md              ← schema da outbox publicado pelo dispatch-engine (de Stage 1)
│   ├── wallet-tables.md            ← ledger, processed_events, voucher, payment_cycle, expiry_alert
│   └── api-spec.yaml               ← OpenAPI da API do wallet (de Stage 3)
├── contracts/
│   └── integration-contract-extension.md  ← patch v1.1.0 do contrato global
├── decisions/
│   └── DECISIONS.md                ← ADRs: G1, G2, G3, G4, Q5–Q14 + flags Q10/Q11/Q13
├── constants/
│   └── payment-rules.md            ← dia=quinta, expiração=6m, bônus=1.10, polling=5s, ttl_outbox=90d
└── references/
    ├── dispatch-engine-pointers.md ← arquivos do dispatch-engine que o wallet consulta (driver model, route states, push.py, auth.py)
    ├── pntrsoft-conventions.md     ← convenções do monorepo (ruff line=100 double, naming en + msgs pt-BR)
    └── icm-patterns.md             ← Patterns 1-15 aplicáveis
```

---

## 7. Variáveis (Placeholders) — Pattern 5

Nomeadas `{{SCREAMING_SNAKE_CASE}}`. Coletadas pelo `setup/questionnaire.md` do workspace wallet.

| Placeholder | Default | Stage que consome | Origem |
|-------------|---------|-------------------|--------|
| `{{PROJECT_NAME}}` | `wallet` | todos | nome do workspace |
| `{{TIMEZONE}}` | `America/Sao_Paulo` | 2, 4, 5 | crítico — sem isso, "quinta-feira" vira bug ao redor da meia-noite UTC |
| `{{POLLING_INTERVAL_SECONDS}}` | `5` | 2 | Q2-residual |
| `{{OUTBOX_TTL_DAYS}}` | `90` | 1, 2 | Q2-residual |
| `{{BALANCE_EXPIRY_MONTHS}}` | `6` | 2, 5 | Q13 |
| `{{EXPIRY_WARNING_DAYS}}` | `30,7` | 5 | Q13 |
| `{{PAYMENT_DAY_OF_WEEK}}` | `thursday` | 2, 4, 5 | Q7 (resolve com `{{TIMEZONE}}`) |
| `{{CONVERSION_BONUS_MULTIPLIER}}` | `1.10` | 2, 4, 5 | Q11 (working) |
| `{{JWT_ACCESS_TTL_MINUTES}}` | `30` | 3, 5 | Stage 3 (renomeado de `{{JWT_TTL_MINUTES}}` para inequívoco) |
| `{{JWT_REFRESH_TTL_HOURS}}` | `24` | 3, 5 | Stage 3 |
| `{{DISPATCH_ENGINE_BASE_URL}}` | `http://localhost:8001` (dev) | 3 | analysis-report |
| `{{WALLET_BASE_URL}}` | `http://localhost:8002` (dev) | 5 | nova porta dev (não conflita com dispatch:8001 nem ordersmonit:8000) |
| `{{FCM_CREDENTIALS_PATH}}` | `_secrets/fcm-wallet.json` | 5 | Q8 (FCM próprio) |
| `{{SUPABASE_DB_URL}}` | `postgresql+asyncpg://...` | 2, 3, 4 | Supabase |
| `{{MANAGER_SESSION_SECRET}}` | (random no setup) | 4 | Q3+Q9 |
| `{{MANAGER_INITIAL_USER_EMAIL}}` | (do questionário) | 4 | bake na bootstrap migration do Alembic — one-shot, sem CLI |
| `{{MANAGER_INITIAL_USER_PASSWORD_HASH}}` | (do questionário, hash argon2) | 4 | idem |

**Hardcoded por escolha (não promovido a placeholder)**: `LEDGER_CURRENCY=BRL`, `VOUCHER_CODE_LENGTH=8`. Comentário no código aponta "facilmente promovido a placeholder se/quando necessário".

**Hardcoded no framework** (não vira placeholder): estrutura de pastas, fluxo de stages, tabelas de checkpoint, padrão de Audit/Verify, idioma português.

**Observabilidade — métricas obrigatórias (sumário)**: `wallet_consumer_lag_seconds`, `wallet_credit_failures_total`, `wallet_ledger_balance_check_passed`, `wallet_auth_attempts_total`, `wallet_auth_latency_seconds`, `wallet_dispatch_validation_failures_total`, `wallet_redemption_attempts_total`, `wallet_pwa_fcm_send_failures_total`, `wallet_pwa_redemption_rate`, `manager_actions_total`, `manager_login_failures_total`, `wallet_rule_applications_total`. Logs estruturados em JSON com `event_id`/`request_id`/`actor_id` em todos os pontos sensíveis. Decisão de stack de métricas (Prometheus? OpenTelemetry? log-based?) fica para Stage 03 — convenção pntrsoft é log estruturado em arquivo + `print()` (mínimo absoluto), mas wallet pode subir o nível.

---

## 8. Scripts de Automação (`_scripts/`)

Pattern 11. Tarefas mecânicas viram script; agente foca em raciocínio.

```
_scripts/
├── verify-stage-1.sh    ← markdownlint contract; sqlfluff outbox-schema.sql
├── verify-stage-2.sh    ← ruff check + format; alembic upgrade head --sql; pytest --cov-fail-under=80; replay idempotency test; ledger balance check
├── verify-stage-3.sh    ← ruff; pytest; spectral lint openapi.yaml; contract tests
├── verify-stage-4.sh    ← ruff; pytest; playwright E2E smoke
├── verify-stage-5.sh    ← eslint; tsc --noEmit; vitest; vite build; railway up --detach (dry)
├── verify-stage-6.sh    ← ruff; pytest; rule simulator dry-run
├── run-all-checks.sh    ← executa 1–5 em sequência (6 sob demanda)
└── outbox-cleanup.sh    ← job noturno: DELETE FROM route_events WHERE consumed_at < NOW() - INTERVAL '90 days'
```

---

## 9. State Tracking — Pattern 10

| Arquivo | Localização | Conteúdo |
|---------|-------------|----------|
| `PROGRESS.md` | root do workspace `wallet` | Fase atual, última sessão, decisões da sessão, próximos passos, padrões de edição recorrentes (espelho do template do builder + convenção do `ecosystem-evolution`) |
| `DECISIONS.md` | `shared/decisions/` | ADR simplificado: data + título + contexto + decisão + consequências + status (`active`/`pending-validation`/`blocked-by-external`) |
| `CHANGELOG.md` | root | Versão semântica conforme stages avançam |

---

## 10. Skills do Workspace

Pattern 4 — skills bundled, copiadas para dentro do workspace.

| Skill | Escopo | Origem |
|-------|--------|--------|
| `/cross-check` | Roda antes de qualquer mudança em arquivo contract-sensitive (lista própria do wallet: `outbox_consumer.py`, `event_mapper.py`, `credit_engine.py`, schema de `wallet_ledger`, schema de `processed_events`, `shared/contracts/integration-contract-extension.md`, **e cross-projeto**: `D:\pntrsoft\dispatch-engine\src\services\route_events_publisher.py`, `D:\pntrsoft\dispatch-engine\src\routers\routes.py:78` e `:123`, migration Alembic da `route_events` no dispatch). Replicado do dispatch-engine. | dispatch-engine (`/cross-check`) |
| `/test` | Roda `verify-stage-N.sh` do stage atual | dispatch-engine |
| `/migrate` | `alembic revision --autogenerate -m "..."` + `alembic upgrade head` (espelho do dispatch-engine) | dispatch-engine |

---

## 11. CI/CD — GitHub Actions

Antecipado para o Stage 02 (não esperar Stage 05). Custo de antecipar é zero, e protege o ledger desde o primeiro commit.

```
.github/workflows/
├── verify.yml          ← Habilitado a partir do Stage 02. Roda em PR e push em main: ruff check + ruff format --check + pytest --cov-fail-under=80
├── build-pwa.yml       ← Habilitado a partir do Stage 05. Roda em PR e main: eslint + tsc + vitest + vite build
└── deploy.yml          ← Habilitado a partir do Stage 05. Roda em push em main: deploy backend Railway + deploy PWA Railway/Vercel + alembic upgrade head em produção
```

Recomendação para retroportar (escopo `ecosystem-evolution`): mesmo `verify.yml` para `dispatch-engine` e `ordersmonit/orders_monitor`. Fora do escopo do workspace `wallet` em si.

---

## 12. Níveis de Automação por Stage — Pattern 14

| Stage | Nível | Justificativa |
|-------|-------|---------------|
| 1 — Discovery & contract | **Colaborativo** | Decisões de produto/contrato — usuário no loop |
| 2 — Wallet core + ingestão | **Review-gate** | Ledger double-entry + idempotência são críticos |
| 3 — API + auth | **Review-gate** | Auth tem implicações de segurança |
| 4 — Painel manager | **Auto** | CRUD com padrão bem definido, baixo risco |
| 5 — PWA + marketplace | **Review-gate** | UX do usuário final + checkout de voucher |
| 6 — Promoções avançadas | **Auto** | Opcional, escopo definido após estabilidade |

---

## 13. Pré-requisitos de Ferramentas — Pattern 7

| Ferramenta | Stage | Obrigatória? | Setup |
|------------|-------|--------------|-------|
| Python 3.12 | 2, 3, 4, 6 | sim | `pyenv install 3.12` |
| Node 20+ | 5 | sim | `nvm install 20` |
| Postgres local (docker-compose) | 2, 3, 4 | sim | reusa pattern do dispatch-engine |
| Supabase project | 2 (prod), 3, 4 | sim em prod | console Supabase |
| Firebase project (FCM próprio) | 5 | sim em prod | console Firebase, novo projeto separado do dispatch-engine |
| Railway project (backend + PWA) | 5 | sim em prod | console Railway |
| `ruff` 0.8.0+ | 2, 3, 4, 6 | sim | em `pyproject.toml` |
| `playwright` | 4, 5 | sim | `playwright install --with-deps chromium` |
| `spectral-cli` | 3 | sim | `npm install -g @stoplight/spectral-cli` |
| `markdownlint-cli2` | 1 | opcional | `npm install -g markdownlint-cli2` |
| `sqlfluff` | 1 | opcional | `pip install sqlfluff` |

---

## 14. Triggers do Workspace

Alinhado com 6/6 dos workspaces ICM existentes (apenas `setup` e `status`) + adição:

| Keyword | Ação |
|---------|------|
| `setup` ou `configurar` | Executa `setup/questionnaire.md` — 1 passada com placeholders + perguntas residuais (JWT TTL, secrets) |
| `status` | Mostra completude dos 6 stages baseado em `stages/*/output/` |
| `cross-check` | Dispara skill `/cross-check` se a tarefa toca arquivo contract-sensitive |

---

## 15. Validação dos Audits do Builder

Verificações do Stage 02 do builder (CONTEXT.md §Audit) sobre este documento:

| Verificação | Status |
|-------------|--------|
| Clareza de stages — todo stage tem responsabilidade única e artefato de output nomeado | ✓ §5 |
| Cadeia input/output — todo input de stage é fornecido pelo usuário ou produzido por stage anterior | ✓ §5 (Stage 1 lê integration-contract + decisões; 2–6 leem stage anterior) |
| Contexto compartilhado — recursos cross-stage listados separados dos específicos | ✓ §6 |
| Cobertura de variáveis — todo detalhe específico do usuário é capturado como variável nomeada | ✓ §7 (14 placeholders) |
| Níveis de automação — todo stage tem nível definido | ✓ §12 |
| Insights da análise refletidos no mapa | ✓ G1/G2/G3/G4/G7/Q14 todos refletidos em §2; Patterns ICM em §6, §8, §10 |

---

## 16. Pendências para o Stage 03 (Mapping) do builder e taxonomia de mudanças futuras

### Pendências para o Stage 03

O Stage 03 do builder vai consumir este documento para formalizar contratos finais. Antes dele, observar:

1. **Q14 destravou G7/CPF.** Schema do `Driver` snapshot no wallet pode fechar agora — não precisa de CPF no MVP.
2. **Q11 working = (c) misto.** Mapping deve gerar schema do ledger com suporte à conversão (entrada `voucher_redeem` no ledger). Se Q11 mudar, ver taxonomia abaixo.
3. **Q10 bloqueia coluna fiscal/contábil específica.** Mapping não precisa de coluna nominal de imposto no ledger; consequência contábil acontece no relatório, não no schema. Bloqueio é parcial — schema fecha; classificação contábil pode esperar.
4. **Decisões `pending-validation` entram no `DECISIONS.md` do Stage 03 do wallet** com status flag, não como decisão definitiva.
5. **Stage 2 do wallet exige patch cross-projeto no dispatch-engine.** O `/cross-check` precisa cobrir os arquivos do dispatch listados em §10. Stage 03 deve reservar uma seção do `contract-extension.md` para descrever a mudança no dispatch (publisher hook, transação atômica, índice da outbox).

### Taxonomia de mudanças futuras (importante para o futuro-você)

Para evitar re-derivar quando uma decisão muda, classificar em duas categorias:

**Triggers de re-discovery (volta para Stage 02 do builder, regerar workflow-map):**
- Q11 muda de (c) para (b) ou (a) — pivô de produto. Em (b) puro elimina: tela "saldo a pagar quinta" no PWA, aba "Pagamentos da quinta" no manager, mecânica do bônus 1.10x, coluna `paid_at`/`status=pending_payment` no ledger, lógica de cut-off semanal. Q10 muda junto: deixa de ser adiantamento de remuneração e fica como benefício/programa de fidelização. Em (a) cria mecanismo de saque (Pix/transferência) — nova integração externa.
- Q14 muda de (b) para (a) — volta a haver integração wallet↔ordersmonit, CPF obrigatório no Driver, NFC-e nominal no resgate, schema fiscal no ledger. Stage 6+.
- Q10 muda enquadramento jurídico do saldo (resultado do contador) — pode exigir colunas fiscais/contábeis no ledger, regime tributário no driver, e refletir no relatório.

**Triggers de ajuste local (mantém workflow-map, atualiza placeholder):**
- Q13 muda período de expiração → ajusta `{{BALANCE_EXPIRY_MONTHS}}`.
- `{{CONVERSION_BONUS_MULTIPLIER}}` muda valor → ajusta config + relatório de impacto.
- Q7 muda dia de pagamento → ajusta `{{PAYMENT_DAY_OF_WEEK}}`.
- `{{POLLING_INTERVAL_SECONDS}}` ajusta cadência → ajusta config.
- `{{OUTBOX_TTL_DAYS}}` ajusta TTL → ajusta job noturno + config.

Quando uma decisão `pending-validation` se confirmar, classificar nesta taxonomia antes de mexer.
