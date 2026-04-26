# Stage 01 — Análise: Wallet (stored value para entregadores)

**Data:** 2026-04-25
**Workspace alvo:** `wallet` (a ser construído via Workspace Builder v2)
**Domínio:** stored value ecosystem para motoboys da hamburgueria, integrado ao dispatch-engine
**Maturidade do projeto wallet:** Greenfield (sem código próprio ainda)
**Maturidade do ambiente em volta:** Mature — dispatch-engine e ordersmonit em produção, integration-contract formal e 6 workspaces ICM ativos no monorepo

> Substituiu um relatório anterior que analisava o dispatch-engine como projeto-alvo. O alvo correto deste workspace builder é o wallet; o dispatch-engine entra aqui como dependência consumida.

---

## 1. Tech Stack Confirmado (via análise das dependências)

### Backend — espelhar exatamente o dispatch-engine (fonte do padrão pntrsoft moderno)

| Componente | Versão / Config | Fonte |
|------------|-----------------|-------|
| Python | 3.12 | `D:\pntrsoft\dispatch-engine\pyproject.toml` |
| FastAPI | 0.115.0+ | idem |
| SQLAlchemy | 2.0 (async) | idem |
| Alembic | 1.14.0 | idem |
| Pydantic | v2 | idem |
| Driver de DB | asyncpg | idem |
| Banco | PostgreSQL via Supabase em prod, docker-compose local | `docker-compose.yml` |
| Linter | Ruff 0.8.0+, `line-length=100`, `quote-style=double`, `target-version=py312` | `pyproject.toml:28-36` |
| Migrations | Alembic, `revision --autogenerate` + `upgrade head` | `alembic/` |
| Auth atual no dispatch-engine | **`auth_code` 6 dígitos via query string** (NÃO JWT) | `src/routers/auth.py` |
| FCM | Firebase via `src/services/push.py`, env `FCM_CREDENTIALS_PATH` | `src/services/push.py` |
| Admin | FastAPI + Jinja2 SSR, **sem autenticação** | `src/routers/admin.py`, `src/templates/` |
| Dockerfile | sim (Python 3.12-slim, multi-stage) | `Dockerfile` |
| CI/CD | **inexistente** — sem `.github/workflows`, sem Makefile | — |

### ordersmonit (referência arquitetural ICM, **não** referência de stack moderno)

| Componente | Versão / Config | Observação |
|------------|-----------------|------------|
| Python | 3.10+ | mais antigo |
| FastAPI | sim | sem `Depends` rigoroso, usa `Request.app.state.config` |
| Pydantic | **não usa** | validação manual em services |
| Linter | **nenhum** | sem ruff/black |
| Config | JSON estático + `_load_config()` por módulo | sem `.env`, sem Pydantic Settings |
| Testes | pytest com `conftest.py` + fixtures | OK, mas básico |
| GUI legada | PyQt6 | irrelevante para wallet |
| CI/CD | inexistente | — |

**Conclusão de stack para o wallet:** seguir o stack do **dispatch-engine** (Python 3.12 + FastAPI + SQLAlchemy 2.0 async + Pydantic v2 + Alembic + ruff `line-length=100` `quote-style=double` + asyncpg + Supabase). O ordersmonit é apenas referência arquitetural (estrutura `routers/` + `services/`, layout de testes, padrão ICM, Import Guard) — não copiar dele decisões de stack.

---

## 2. Mapa de Estrutura

### Monorepo `D:\pntrsoft\` (contexto do wallet)

```
D:\pntrsoft\
├── CLAUDE.md                              (orquestrador raiz, lista 6 workspaces ICM)
├── CONTEXT.md
├── REFERENCES.md
├── Interpreted-Context-Methdology/        (framework ICM clonado)
├── dispatch-engine/                       (sistema de rotas — fonte de driver/route para o wallet)
├── ordersmonit/orders_monitor/            (captura de pedidos — irrelevante para wallet, exceto via contrato global)
├── integration-contract/CONTRACT.md       (v1.0.0 — contrato ordersmonit↔dispatch-engine; precisa extensão para wallet)
├── plans/
└── workspaces/
    ├── dispatch-engine-build/             (5 stages, inglês)
    ├── admin-orders-pull/                 (3 stages, português)
    ├── orders-dispatch-integration/       (4 stages, português)
    ├── ordersmonit-web-migration/         (5 stages, inglês)
    ├── dispatch-engine-p0-fixes/          (3 stages, português)
    └── ecosystem-evolution/               (5 stages, português, único com PROGRESS.md)
```

### dispatch-engine (profundidade 3) — superfície que o wallet vai consumir

```
D:\pntrsoft\dispatch-engine\
├── CLAUDE.md
├── .ai-context/
│   ├── backend.md, integrations.md, admin-panel.md, infra.md, testing.md, docs.md, pwa.md
│   └── playbooks/  (bugfix.md, new-feature.md, migration.md, refactor-safe.md)
├── src/
│   ├── main.py, config.py, database.py
│   ├── models/      (driver.py, route.py, stop.py, geocache.py)   ← FONTE para FK lógica do wallet
│   ├── routers/     (auth.py, routes.py, admin.py, events.py, orders_pull.py, optimize.py, drivers.py, geocode.py)
│   ├── services/    (push.py — FCM; orchestrator.py — VRP; geocoding.py; routing.py; optimizer.py)
│   ├── schemas/     (Pydantic v2)
│   ├── templates/   (Jinja2: base, dashboard, routes, drivers, route_detail)
│   ├── static/, utils/
├── alembic/, tests/, docs/
├── pwa/             (React + Vite + TS — referência de padrão para o PWA do wallet)
├── Dockerfile, docker-compose.yml, pyproject.toml, alembic.ini, .env.example
```

---

## 3. Modelos do dispatch-engine que o Wallet vai consumir

### Driver (`D:\pntrsoft\dispatch-engine\src\models\driver.py`)

| Campo | Tipo | Observação para o wallet |
|-------|------|--------------------------|
| `id` | UUID string (36 chars) | **PK lógica usada como FK no wallet** |
| `name` | String(100) | snapshot opcional no wallet (relatórios) |
| `phone` | String(20) | snapshot opcional |
| `plate` | String(10) | irrelevante para wallet |
| `max_capacity` | int (default 10) | irrelevante |
| `status` | Enum(`online`/`offline`) | irrelevante |
| `push_token` | String(255) | **FCM token — wallet pode reaproveitar via dispatch-engine, não duplicar** |
| `auth_code` | String(6), unique | **mecanismo de identidade atual — base para definir auth do wallet** |
| `created_at`, `updated_at` | datetime | |

**Sem CPF.** Decidir no Stage 02 se o wallet precisa de CPF para emissão fiscal de itens trocados, ou se trata apenas saldo interno.

### Route (`D:\pntrsoft\dispatch-engine\src\models\route.py`)

- Estados: `planned` → `in_progress` → `completed` | `cancelled`
- **Sem log de transições** (status só sobrescreve no campo)
- **Não há evento emitido na transição → `completed`**
- Pontos de transição:
  - `src/routers/routes.py:56-87` (manual via `PATCH /routes/{route_id}/status`)
  - `src/routers/routes.py:123` (auto-complete quando todos os stops viram `delivered`/`failed`)

---

## 4. Patterns Existentes Detectados

### Patterns positivos a reaproveitar

- **Layout `routers/` + `services/` + `schemas/` + `models/`** — padrão FastAPI moderno em uso no dispatch-engine; copiar literalmente.
- **`AsyncSession` injetado via `Depends(get_db)`** — wallet usará igual.
- **Pydantic v2 schemas separados de SQLAlchemy models** — evitar exposição descontrolada via `from_attributes`.
- **Naming**: variáveis/funções/classes em inglês; mensagens e logs em português BR.
- **Docs em `.ai-context/`** com `backend.md`, `integrations.md`, `infra.md`, `playbooks/` — wallet replica essa pasta.
- **Skills do dispatch-engine** (`/test`, `/migrate`, `/cross-check`) — wallet mantém pelo menos `/cross-check` para mudanças contract-sensitive.
- **Workspaces ICM (6/6)**: `setup/questionnaire.md` em todos, `shared/` em 5/6, `stages/NN-name/{CONTEXT.md, references/, output/}` em todos, tabela "What to Load / Do NOT Load" em 5/6. Português é majoritário (4/6). Sem emojis. Tom técnico-imperativo.
- **Integration Contract** (CONTRACT.md): formato numerado, "Mudanças breaking" listadas, changelog versionado, lista de "Arquivos contract-sensitive" — wallet estende, não cria contrato paralelo.

### Patterns que **não** existem ainda — wallet introduz ou exige extensão

- **Auth com JWT**: dispatch-engine não tem. Decidir Stage 02/03: (a) wallet emite seu próprio JWT após validar `auth_code` contra dispatch-engine, ou (b) dispatch-engine ganha endpoint `/driver/token` que devolve JWT. Q4 do questionário assumiu JWT pré-existente — **isso é gap, não fato**.
- **Webhook out / event publishing**: dispatch-engine não publica nada. Wallet exige que dispatch-engine ganhe hook em `routes.py` (auto-complete e manual) chamando `services/events.py::on_route_completed()`, OU que o `/events/routes` SSE passe a emitir `completed` (mudança no SSE atual).
- **Idempotência em ingestão**: wallet precisa de `event_id` + ledger imutável; dispatch-engine não fornece event id hoje.
- **Auth no admin Jinja2**: o admin do dispatch-engine é público. Wallet precisa decidir auth do painel manager (sessão? basic auth? JWT separado?).
- **CI/CD versionado**: nenhum projeto pntrsoft tem `.github/workflows`. Wallet introduz GitHub Actions. Recomendação: retroportar via `ecosystem-evolution` em fase futura.
- **Pre-commit hooks**: ausentes em todos os projetos. Wallet pode adotar (ruff format + ruff check + pytest rápido) — opcional.

---

## 5. Integration Contract (estado atual)

`D:\pntrsoft\integration-contract\CONTRACT.md` (v1.0.0, 2026-03-27):

- Cobre **apenas** ordersmonit ↔ dispatch-engine (endpoints `/integracao/rotas`, `/integracao/pedidos`, schema `OrderPayload`).
- **Não cobre** dispatch-engine ↔ wallet — exigirá extensão das Seções 2 (API Contract), 3 (Modelos compartilhados), 7 (Changelog) e 8 (Arquivos contract-sensitive).
- Convenção mantida: secção numerada, "Mudanças breaking" listadas, changelog versionado, arquivos contract-sensitive identificados, `/cross-check` antes de mudar.
- **Stage 03 (Mapping) do wallet produz a extensão deste arquivo** — não criar contrato paralelo nem documento separado.

---

## 6. Maturidade

| Categoria | Justificativa |
|-----------|---------------|
| **Greenfield (para o wallet)** | Sem código próprio. Será construído do zero. |
| **Mature (para o ambiente em volta)** | dispatch-engine tem `.ai-context/` completo, playbooks, hooks ICM, integration-contract formal, 6 workspaces ICM ativos no monorepo, Alembic, Dockerfile, ruff, Pydantic v2. Único gap: CI/CD versionado. |

---

## 7. Gaps e Decisões Fixadas no Checkpoint

Decisões fixadas pelo dono do produto no checkpoint do Stage 01 (2026-04-25). Estas reduzem o escopo do Discovery — o Stage 02 não as reabre.

| # | Tema | Decisão / Status |
|---|------|------------------|
| **G1** | Auth cross-system | **FIXADO**: o entregador faz login no wallet com o **mesmo `auth_code` 6 dígitos** do dispatch-engine (UX consistente), e o wallet **emite seu próprio JWT curto** após a troca. Dispatch-engine permanece intocado — `auth_code` continua válido para login. Wallet vira a primeira peça do ecossistema com auth de verdade. Razão: não propagar o anti-padrão do `auth_code` em query string; permitir que o dispatch-engine adote JWT depois sem breaking change. |
| **G2** | Evento `route.completed` | **FIXADO**: **outbox pattern no dispatch-engine**, não webhook saindo. Nova tabela `route_events` no schema do dispatch, wallet faz polling/CDC. Razão: idempotência (G3) cai naturalmente do `event_id` da outbox; evita criar infra de retry/DLQ HTTP que ainda não existe. |
| **G3** | Idempotência | **FIXADO**: `event_id` vem da outbox `route_events` (consequência direta de G2). Wallet usa esse id como chave única no ledger. |
| **G4** | Auth do manager | **FIXADO**: manager **exige auth de verdade desde o dia 1** — não replicar o admin público do dispatch-engine. Manager mexe com dinheiro real e regras de remuneração. Decisão de framework (Jinja2 + sessão vs SPA + JWT) fica para o Stage 02; o requisito de auth está cravado. |
| G5 | CI/CD | Em aberto: wallet introduz GitHub Actions (Stage 05). Pode ser retroportado para outros projetos via `ecosystem-evolution` em fase futura. |
| G6 | Extensão do integration-contract | Em aberto: Stage 03 (Mapping) produz a extensão. Convenção do contrato preservada. |
| **G7** | CPF no driver | **PERGUNTA BLOQUEANTE para o Stage 02**: depende de como a hamburgueria emite NFC-e quando o entregador resgata um item. Se resgate emite cupom fiscal nominal → CPF obrigatório (adicionar ao Driver no dispatch-engine OU capturar no onboarding do wallet). Resposta determina o schema — não dá para começar Stage 03 sem decidir. |
| G8 | FCM | Em aberto: reusar `services/push.py` do dispatch-engine via import, duplicar, ou wallet ter o seu? Stage 02 decide. |

---

## 8. Padrões para o Workspace Wallet (informa Stages 04 e 06)

Baseado na análise dos 6 workspaces ICM existentes, o workspace wallet deve:

- **Idioma**: **português** (alinhado com 4/6 workspaces, e este onboarding foi conduzido em português).
- **Triggers**: limitar a `setup` + `status`. Considerar `/cross-check` (alinhado com convenção do dispatch-engine) se houver mudança em arquivo contract-sensitive.
- **Estrutura**: `setup/questionnaire.md` + `shared/` (referências, decisões, contrato local) + `stages/NN-name/{CONTEXT.md, references/, output/}`.
- **Tabelas**: markdown `| | |` sem emojis, com seção "What to Load / Do NOT Load" por stage.
- **PROGRESS.md**: opcional — adicionar se a 6ª stage opcional (engine de promoções avançadas) virar ciclo separado pós-MVP.
- **Tom**: técnico-didático, imperativo, paths absolutos sempre que possível.
- **6 stages propostos** (último opcional) — ver questionário Q6.

---

## 9. Perguntas para o Stage 02 (Discovery)

Q1–Q9 originais (algumas já cortadas pelas decisões fixas em §7) + Q10–Q14 adicionadas no checkpoint para cobrir o coração do produto que o questionário inicial não tocou.

### Operacionais / técnicas (refinadas)

1. ~~**Auth cross-system**~~ — **FIXADO em G1**.
2. ~~**Mecanismo de evento `route.completed`**~~ — **FIXADO em G2** (outbox + polling). O que resta: **frequência do polling** (3s como o SSE existente? maior?) e **estratégia de cleanup** da `route_events` (TTL? marca como `consumed`?).
3. **Auth do painel manager**: Jinja2 + sessão server-side, ou SPA React + JWT separado? (G4 fixou que TEM auth; falta escolher o como.)
4. ~~**Idempotência**~~ — **FIXADO em G3** (event_id da outbox).
5. **Marketplace — escopo MVP**: catálogo apenas hamburgueria? Inventário com estoque ou só vouchers? (Emissão fiscal foi para Q14 abaixo.)
6. **Regras de remuneração — granularidade**: por entrega (flat), por rota (média), ou regras compostas (bônus por dia/quantidade/tempo) desde o MVP? (Q6 do questionário deixou a 6ª stage opcional para regras avançadas — confirmar que MVP é flat.)
7. **Ciclo de pagamento**: dia fixo da semana? Saldo trava antes do pagamento? Marketplace consome saldo "disponível" ou "consolidado"? (Liga com Q13 — expiração.)
8. **FCM**: wallet reusa `services/push.py` do dispatch-engine via import, duplica, ou tem seu próprio?
9. **Painel manager — framework**: FastAPI + Jinja2 (consistente com dispatch-engine) ou React?

### Coração do produto (adicionadas no checkpoint)

10. **Natureza jurídica do saldo**: motoboy é MEI/PJ ou autônomo? Saldo é **adiantamento de remuneração** (passivo trabalhista/comercial) ou **benefício/bônus** (despesa)? Muda contabilização e potencialmente tributação.
11. **Saldo é sacável em dinheiro?** Se sim → é "pagamento adiantado com vitrine" (modelo de adiantamento). Se não → é **stored value puro** (modelo Starbucks). Resposta muda enquadramento regulatório (BACEN trata arranjo de pagamento fechado diferente de adiantamento salarial).
12. **Fungibilidade no checkout do marketplace**: resgate é "tudo ou nada" pelo saldo, ou aceita **split** (metade saldo, metade dinheiro/Pix)? Split exige integração de pagamento; tudo-ou-nada não.
13. **Expiração e breakage**: saldo expira? Em quanto tempo? Sem expiração → passivo eterno crescente no balanço. Com expiração → precisa comunicação clara + tela "saldo a expirar" no app.
14. **Resgate gera pedido downstream?** Quando o entregador troca saldo por um burger, isso vira pedido no ordersmonit/POS (entra na fila normal de produção) ou é fluxo paralelo (retirada manual no balcão)? Define se há **terceira integração (wallet ↔ ordersmonit)** ou só duas (wallet ↔ dispatch-engine). Inclui também **emissão fiscal**: se NFC-e nominal → CPF obrigatório (G7).

### Bloqueio explícito de Stage

- **G7 + Q14** bloqueiam o Stage 03 (Mapping). Sem decidir emissão fiscal e CPF, não dá para fechar schema do `Driver` snapshot no wallet nem para escrever a extensão do contrato.

---

## 10. Saída deste Stage

Este relatório é o input do Stage 02 (`stages/02-discovery/CONTEXT.md`). Aguardando aprovação no checkpoint antes de prosseguir.
