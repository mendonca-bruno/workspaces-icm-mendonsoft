# Decisions

ADRs (Architecture Decision Records) do workspace. Cada decisão tem: contexto, escolha, consequências, status, gatilho de revisão.

Status possíveis: `active` | `active-permanent` | `pending-validation` | `superseded-by-<ADR>` | `with-review-checkpoint`.

---

## ADR-001 — KB hierárquica em vez de monolítica

**Status:** active
**Data:** 2026-04-26

**Contexto:** Workspace anterior tinha `kb/<area>.md` monolítico (1 arquivo grande por área de tecnologia, agregando vários projetos). Buscas por keyword falhavam porque o agente carregava o arquivo inteiro e não tinha pivôs semânticos.

**Decisão:** Estruturar KB como `kb/<tecnologia>/<sistema>/<dossie>.md`, com `CONTEXT.md` em cada nível roteando para baixo. Cada dossiê é coeso (1 schema, 1 fluxo, 1 conjunto de APIs relacionadas), nunca > ~400 linhas.

**Consequências:** Roteamento por carregamento seletivo passa a funcionar (paper ICM §recursive routing). Custo: migração inicial dos `.md` monolíticos antigos. Mitigado pela Fase B do plano via overlay paralelo.

**Trigger de revisão:** Se algum dossiê passar de 600 linhas duas vezes seguidas, repensar a granularidade.

---

## ADR-002 — Sources canônicos em `kb/_sources/`, dossiês como destilação

**Status:** active

**Contexto:** Q1 do plano. Manter DDL/XML/JIL em formato `.md` significa duplicar a fonte. Mas anotar a fonte original com metadados ICM polui o source canônico (que precisa ser fiel ao banco).

**Decisão:** Sources brutos vivem em `kb/_sources/<tech>/<sistema>/<arquivo-original>` (L4 — fiel à origem). Dossiês em `kb/<tech>/<sistema>/<dossie>.md` (L3) são destilação semântica: relações FK, joins críticos, regras inferidas, padrões de uso. Frontmatter do dossiê referencia: `source + source_sha256 + last_distilled`. Stage 06 detecta drift comparando hash atual com armazenado.

**Consequências:** Vencer trade-off destilação vs. fidelidade. Stage 06 precisa rodar semanal para evitar dossiês defasados.

**Trigger de revisão:** Se >30% dos dossiês ficarem com `last_distilled` > 180 dias, repensar cadência de manutenção.

---

## ADR-003 — Cases como cidadãos privilegiados, separados da KB

**Status:** active

**Contexto:** Q3 do plano. Workspace anterior misturava casos resolvidos no `.md` da área. Cases perdiam relevância privilegiada na busca.

**Decisão:** Diretório `cases/` separado de `kb/`, com 1 arquivo por caso e frontmatter tipado (`symptoms`, `systems_involved`, `root_cause`, `resolution_query`, `resolution_steps`, `reuse_count`, `promoted_to`). Stage 03 consulta `cases/INDEX.md` ANTES de qualquer dossiê de KB. Stage 05 promove padrões recorrentes (≥3 cases similares) a `kb/_playbooks/`.

**Consequências:** Resolução de tickets repetidos vira lookup direto. Custo: Stage 05 adiciona ~5min por ticket fechado para preencher frontmatter — aceito como investimento.

**Trigger de revisão:** Se reuse_count médio dos top-10 cases não subir após 2 meses de uso, indicar que o matching no Stage 03 está fraco.

---

## ADR-004 — Stage 02 (clarify) é obrigatório para tickets vagos, com human edit gate

**Status:** active

**Contexto:** Q2 do plano e gap explícito: tickets abertos demais. Workspace anterior improvisava resposta sem clarificar.

**Decisão:** Stage 02 calcula score de clarificação (4 dimensões: sistema identificado, sintoma normalizado, janela temporal, identificador afetado). Se score < threshold, gera `02-clarification.md` com perguntas estruturadas e PARA o pipeline. Bruno cola resposta do solicitante; loop até score OK.

**Consequências:** ~20-40% dos tickets vão pausar na clarificação (estimativa). Latência maior nesses casos, qualidade do plano de ação muito melhor. Alinhado ao paper ICM (heavy-edit stage 1, U-shape).

**Trigger de revisão:** Se < 10% dos tickets disparam clarificação, threshold está baixo demais (agente está sendo permissivo). Se > 60%, threshold está alto (agente exige perfeição).

---

## ADR-005 — Migração via overlay paralelo + APPROVED.md

**Status:** active (uma única vez)

**Contexto:** Plano aprovado em 2026-04-26. Workspace alvo está no ambiente de trabalho do Bruno (não local).

**Decisão:** Usar padrão `workspace-retrofitter/stages/05-apply` — gerar overlay paralelo `<workspace>-icm-overlay/`, com `MIGRATION-PLAN.md` + `file-mapping.csv`; aplicar in-place só após `APPROVED.md`; gravar `rollback-manifest.json` antes de cada move para reverter se necessário.

**Consequências:** Zero risco destrutivo na migração. Custo: passada de 2-3 dias para Bruno revisar o overlay.

**Trigger de revisão:** Não — decisão one-shot. Após go-live, mover `_migration/` para `_archive/` do meta-vault.
