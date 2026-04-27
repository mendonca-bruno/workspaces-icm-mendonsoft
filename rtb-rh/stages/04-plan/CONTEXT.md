# Stage 04: Geração do Plano de Ação

Sintetizar plano executável com queries SQL específicas, validações de log, e (se necessário) scripts BW/PWC/Control-M. Cada passo cita a fonte do contexto (`<dossier#section>` ou `<case-id>`).

## Inputs

| Fonte | Arquivo/Localização | Escopo | Por quê |
|-------|---------------------|--------|---------|
| Ticket clarificado | `../../tickets/<id>/02-clarified.md` | Frontmatter + corpo | Definição do problema |
| Resultado da busca | `../../tickets/<id>/03-search-result.md` | Arquivo completo | Cases + dossiês + playbooks identificados |
| Dossiês citados | Paths em `03-search-result.md` | Apenas seções relevantes | Schemas, joins, regras |
| Playbooks citados | `../../kb/_playbooks/<sintoma>.md` | Arquivo completo | Padrão de resolução conhecido |
| Caso citado (se houve hit forte) | `../../cases/INC<id>.md` | Frontmatter + `resolution_query` + `resolution_steps` | Receita pronta para adaptar |
| Template de plano | `../../reference/plan-template.md` | Arquivo completo | Estrutura do output |
| Patterns SQL | `../../reference/sql-patterns-by-area.md` | Seção da área | Padrões idiomáticos |
| Playbook de logs | `../../reference/log-validation-playbook.md` | Arquivo completo | Como validar logs em cada stack |

## Processo

1. Ler `02-clarified.md` e `03-search-result.md`.
2. Decidir **estratégia**:
   - Se há `case_match` com `resolution_query`: adaptar a query (substituir matrícula/data/etc.) e adaptar `resolution_steps`. Esta é a rota de menor risco.
   - Se há playbook matching: seguir o passo-a-passo.
   - Caso contrário: derivar do zero usando dossiês.
3. **Compor as queries SQL** usando tabelas/joins concretos do dossiê:
   - Cada query precedida por comentário `-- fonte: <dossier#section>`.
   - Queries de **diagnóstico** primeiro (SELECTs) — sempre seguros.
   - Queries de **fix** (UPDATE/DELETE/INSERT) só após diagnóstico ter confirmado o problema.
4. **Compor validações de log** seguindo `log-validation-playbook.md`:
   - Caminho do arquivo de log.
   - Padrão a buscar (regex ou string).
   - O que esperar (ausência ou presença).
5. **Compor passos não-SQL** (se aplicável): re-run de job Control-M, replay de processo BW, redeploy, etc.
6. **Marcar premissas explícitas** em seção `## Premissas`. Se uma premissa for falsa, o plano precisa ser revisado.
7. **Identificar pontos destrutivos**: marcar passos com `[DESTRUTIVO]` quando envolvem UPDATE/DELETE/INSERT em tabela de produção, ou re-run de job que afeta dados.
8. **[Checkpoint — Bloqueante se há `[DESTRUTIVO]`]** Apresentar o plano completo ao Bruno. Bruno aprova/edita/rejeita antes de qualquer execução em produção.
9. Emitir `04-action-plan.md` no formato do `plan-template.md`.

## Checkpoints

| Após Passo | Agente Apresenta | Humano Decide |
|------------|-------------------|---------------|
| 8 | Plano completo com diagnóstico + fix + validação + premissas + passos destrutivos marcados | Aprovar; ajustar; rejeitar e voltar para Stage 03 |

## Verify

| Verificação | Comando | Condição de Aprovação |
|-------------|---------|----------------------|
| Citações resolvíveis | Para cada `<dossier#section>` no plano, o dossier existe e tem a section | grep |
| Tabelas existem em dossiê | Cada tabela em SELECT/UPDATE/etc. aparece em `entities:` de algum dossiê citado | grep |
| Queries diagnóstico antes de fix | Ordem dos passos coloca SELECTs antes de UPDATE/DELETE | inspeção |
| `[DESTRUTIVO]` marcados | Todo UPDATE/DELETE/INSERT/job re-run tem o tag | grep |

## Audit

| Verificação | Condição de Aprovação |
|-------------|----------------------|
| Premissas explícitas | Seção `## Premissas` existe e lista o que precisa ser verdade para o plano funcionar |
| Caminho de log indicado | Toda validação de log indica path completo do arquivo |
| Plano executável por terceiro | Outro membro do time conseguiria seguir o plano sem precisar fazer perguntas adicionais |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Plano de ação | `../../tickets/<id>/04-action-plan.md` | Markdown com seções: `## Resumo`, `## Premissas`, `## Diagnóstico (queries SELECT)`, `## Fix (queries DESTRUTIVAS)`, `## Validação pós-fix`, `## Rollback`, `## Fontes citadas` |
