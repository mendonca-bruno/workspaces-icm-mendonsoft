# Stage 05: Pós-mortem (arquivar caso, promover padrão a playbook)

Após Bruno executar o plano e fechar o ticket, este stage transforma o ticket em **memória organizacional**: arquiva como caso tipado em `cases/`, ou (se há recorrência) propõe promover o padrão a `kb/_playbooks/`.

## Inputs

| Fonte | Arquivo/Localização | Escopo | Por quê |
|-------|---------------------|--------|---------|
| Plano de ação | `../../tickets/<id>/04-action-plan.md` | Arquivo completo | Base do que foi feito |
| Outcome real | Bruno cola em `tickets/<id>/05-outcome.md` | Texto livre | Resultado real (resolveu, parcial, não resolveu) + observações |
| Index de cases | `../../cases/INDEX.md` | Tabela completa | Detectar recorrência (≥3 cases similares) |
| Template de caso | `../../reference/case-template.md` | Arquivo completo | Schema do `cases/INC*.md` |
| Template de playbook | `../../reference/playbook-template.md` | Arquivo completo | Schema do `kb/_playbooks/*.md` |
| Ticket clarificado | `../../tickets/<id>/02-clarified.md` | Frontmatter | Sintomas e sistemas para o frontmatter do caso |

## Processo

1. Ler outcome do Bruno em `05-outcome.md`. Confirmar status: `resolved` | `partial` | `not_resolved`.
2. Extrair **root cause** (1-3 frases) e **resolução final** (queries que efetivamente funcionaram, podem diferir do plano original).
3. **Compor frontmatter do caso** seguindo `case-template.md`:
   - `ticket_id`, `created`, `area`, `systems_involved` (de `02-clarified.md`)
   - `symptoms` (de `02-clarified.md`)
   - `entities` (das queries que rodaram)
   - `root_cause`, `resolution_query`, `resolution_steps`
   - `dossiers_cited` (do `04-action-plan.md`)
   - `duration_min` (Bruno marca)
   - `reuse_count: 0`, `promoted_to: null`
4. **Detectar recorrência**: filtrar `cases/INDEX.md` por `(area, systems_involved, symptoms)` ≈ deste ticket.
   - Se total (incluindo este novo) ≥ 3 nos últimos 90 dias E nenhum tem `promoted_to`: **candidato a promoção**.
5. **[Checkpoint — Decisão de tipagem]**:
   - **Caso singular** (sem recorrência ou recorrência já promovida): salvar em `cases/INC<id>.md`.
   - **Recorrência detectada**: apresentar lista de cases similares + proposta de playbook em `kb/_playbooks/<sintoma>.md`. Bruno decide:
     - Promover agora → executar passo 6.
     - Adiar (registra apenas o caso individual) → marcar entrada em `kb/_pending-updates.md` para Stage 06 reconsiderar.
6. **Promoção a playbook** (se aprovada):
   - Criar `kb/_playbooks/<sintoma>-<sistema>.md` seguindo `playbook-template.md`: abstrai matrículas/datas/IDs específicos, mantém o padrão de resolução.
   - Marcar todos os cases-fonte (incluindo este) com `promoted_to: ../kb/_playbooks/<arquivo>.md` no frontmatter.
   - Atualizar `kb/INDEX.md` (ou flagar para Stage 06).
7. **Detectar contradição com dossiê**: se a resolução real revelou que o dossiê estava errado (ex.: regra documentada não corresponde ao comportamento real), abrir flag em `kb/_pending-updates.md` com `dossier:`, `inconsistency:`, `evidence: tickets/<id>/`.
8. Atualizar `cases/INDEX.md` (ou rodar Stage 06 depois).
9. Se outcome = `not_resolved`: ainda salva o caso (com `resolution_steps:` documentando o que foi tentado), com flag `escalated: true`. Pode virar input para nova rodada de busca.

## Checkpoints

| Após Passo | Agente Apresenta | Humano Decide |
|------------|-------------------|---------------|
| 5 | Frontmatter draft do caso + (se aplicável) lista de cases similares e proposta de playbook | Promover já, adiar promoção, ou apenas arquivar singular |

## Verify

| Verificação | Comando | Condição de Aprovação |
|-------------|---------|----------------------|
| Frontmatter completo | Inspecionar `cases/INC<id>.md` | Campos obrigatórios do `case-template.md` preenchidos |
| `resolution_query` não-vazia se resolved | grep | Para `status: resolved`, campo é não-vazio |
| Cases-fonte marcados | Após promoção, todos os cases listados têm `promoted_to:` | grep |
| Sintomas canônicos | `symptoms:` no caso usa `id`s do `kb/_taxonomies/symptoms.md` | grep |

## Audit

| Verificação | Condição de Aprovação |
|-------------|----------------------|
| Caso reflete a resolução real | `resolution_query` é o que rodou, não o plano original (que pode ter sido revisado) |
| Lições capturadas | Seção `## Lições` preenchida (mesmo que curta) |
| Pending-updates registrado | Se houve contradição com dossiê, `kb/_pending-updates.md` tem entrada |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Caso arquivado | `../../cases/INC<id>.md` | Markdown com frontmatter (schema do `case-template.md`) |
| Playbook (se promoção) | `../../kb/_playbooks/<sintoma>-<sistema>.md` | Markdown (schema do `playbook-template.md`) |
| Cases-fonte atualizados | Frontmatters dos cases promovidos | Campo `promoted_to:` preenchido |
| Pending-updates (se contradição) | `../../kb/_pending-updates.md` | Append no arquivo |
| Index atualizado | `../../cases/INDEX.md` | Tabela markdown |
