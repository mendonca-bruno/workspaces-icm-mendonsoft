# Stage 00: Pre-flight

Avaliar se o source é ICM-compliant. Se não, invocar `workspace-retrofitter` automaticamente. Bloquear pipeline até source estar pronto.

## Inputs

| Fonte | Arquivo / Localização | Escopo | Por quê |
|-------|----------------------|--------|---------|
| Setup | `../../setup/questionnaire.md` (versão preenchida) | Seção `## Suas respostas` | Caminho do source, restrições |
| Config | `../../_config/preflight-policy.md` | Heurística completa | Critérios de PASS/GATE |

## Process

1. Validar que `source_path` existe e é diretório. Abortar se não.
2. Avaliar os 5 marcadores ICM (ver `preflight-policy.md`):
   - `CLAUDE.md` na raiz com seções esperadas
   - `CONTEXT.md` com routing table
   - Stages/silos numerados ou padrão equivalente
   - Camada L3 (`_config/`, `references/` ou `docs/`)
   - `setup/questionnaire.md` ou equivalente
3. Contar marcadores ausentes.
4. Se 0–2 ausentes: **PASS**. Escrever `preflight-report.md` com tabela e seguir para stage 01.
5. Se 3–5 ausentes: **GATE**.
   1. Verificar existência de `../../../workspace-retrofitter/`. Se ausente, abortar com mensagem clara.
   2. Preencher `<retrofitter>/setup/questionnaire.md` herdando dados do questionnaire do polyglot-porter.
   3. Acionar pipeline retrofitter: stages 01 → 04. Pausar antes de stage 05.
   4. Aguardar criação de `APPROVED.md` no overlay do retrofitter (intervenção humana).
   5. Acionar stage 05 (apply) do retrofitter.
   6. Re-avaliar os 5 marcadores no source.
   7. Se ainda 3+ ausentes, escalar (escrever `preflight-report.md` com `STATUS: BLOCKED` e instruir intervenção manual).
6. Escrever `preflight-report.md` com tabela final, decisão, e log da invocação do retrofitter (se aplicável).

## Checkpoints

Sem gate humano em PASS direto. Em GATE, herda os checkpoints internos do retrofitter (após classify e após generate).

## Audit

| Verificação | Aprovação |
|-------------|-----------|
| `source_path` existe | Pasta acessível |
| Decisão registrada | `STATUS: PASS` ou `STATUS: BLOCKED` em `preflight-report.md` |
| Re-avaliação pós-retrofitter (se GATE) | Marcadores ≤ 2 ausentes |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Relatório de preflight | `output/preflight-report.md` | Tabela de marcadores + decisão + log retrofitter |

## Hand-off

Stage 01 lê apenas `output/preflight-report.md` (para confirmar PASS) e o source diretamente.
