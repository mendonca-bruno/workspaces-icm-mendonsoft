# Pre-flight Policy — Avaliação de ICM-Compliance

Política aplicada pelo `stages/00-preflight/`. Decide se o source pode prosseguir direto para stage 01 ou se precisa passar pelo `workspace-retrofitter` antes.

## Heurística (5 marcadores)

Cada marcador AUSENTE conta 1 ponto. Soma final determina ação.

| # | Marcador | Como detectar |
|---|---|---|
| 1 | `CLAUDE.md` na raiz do source | Arquivo existe e tem ≥ 50 linhas com seções "Mapa de Pastas" / "Folder Map" e routing table |
| 2 | `CONTEXT.md` de roteamento | Arquivo existe na raiz com tabela "Para Onde Ir" / "Routing" |
| 3 | Stages/silos numerados | Pasta `stages/`, `silos/`, ou ≥ 3 pastas com prefixo `01-`, `02-`, `03-` etc. |
| 4 | Camada L3 presente | Existe `_config/`, `references/` ou `docs/` com material persistente |
| 5 | Setup/onboarding | Pasta `setup/` com `questionnaire.md` ou equivalente |

## Decisão

| Pontos ausentes | Ação |
|---|---|
| 0–2 | **PASS** — prosseguir para stage 01. Registrar marcadores ausentes em `preflight-report.md` como dívida (não bloqueante). |
| 3–5 | **GATE** — invocar `workspace-retrofitter` no source automaticamente. Bloquear até retrofitter completar (com APPROVED.md) e re-avaliar. |

## Invocação automática do retrofitter

Sequência quando GATE acionado:

1. Verificar se `../workspace-retrofitter/` existe (path relativo ao polyglot-porter). Se não, **abortar** com mensagem instruindo o usuário a clonar/restaurar o retrofitter ou rodar manualmente.
2. Preencher `../workspace-retrofitter/setup/questionnaire.md` automaticamente com:
   - `target_path` = source path do polyglot-porter
   - `purpose` = "Source code project to be migrated by polyglot-porter"
   - `audience` = "solo+ai"
   - `prior_organization` = "preserve" (não destruir estrutura existente do source)
   - `ignore_paths`, `sensitive_patterns` herdados do questionnaire do polyglot-porter
3. Acionar pipeline retrofitter: stages 01 → 04. Pausar antes de stage 05 (apply) — exigir intervenção humana para criar `APPROVED.md` no overlay do retrofitter.
4. Quando o retrofitter completar stage 05 (apply), re-avaliar os 5 marcadores no source. Se ainda < 3 ausentes, prosseguir; senão escalar.

## Edge cases

- **Retrofitter ausente.** Mensagem: "workspace-retrofitter não encontrado em `../workspace-retrofitter/`. Polyglot-porter requer ICM-compliance no source. Restaure o retrofitter ou estruture o source manualmente conforme [`_core/CONVENTIONS.md`]."
- **Source já tem ICM mas com nomes não-padrão.** Se CLAUDE.md/CONTEXT.md existem mas pastas usam outro padrão (ex: `phase-01/` em vez de `01-phase/`), conta como marcador 3 PRESENTE. Anotar em `preflight-report.md` mas não bloquear.
- **Source é monorepo.** Aplicar heurística por subprojeto. Apenas o subprojeto a ser migrado precisa passar.
- **Source vazio ou inexistente.** Abortar com erro antes mesmo de avaliar marcadores.

## Output

`stages/00-preflight/output/preflight-report.md` contém:

- Tabela com cada marcador e PASS/FAIL
- Pontuação total
- Decisão (PASS ou GATE)
- Se GATE: log da invocação do retrofitter, link para `<source>-icm-overlay/MIGRATION-PLAN.md`
- Re-avaliação pós-retrofitter (se aplicável)
