# Template — Caso resolvido

Use este molde ao arquivar um ticket em `cases/INC<numero>.md`. Frontmatter é **obrigatório** — Stage 05 (Verify) rejeita arquivo incompleto.

```markdown
---
ticket_id: INC1234567                            # número canônico do ticket no sistema de origem
created: 2026-04-26                              # data da resolução (YYYY-MM-DD)
duration_min: 45                                 # tempo total gasto (opcional mas recomendado)
status: resolved                                 # resolved | partial | not_resolved | escalated
area: folha                                      # folha | ponto | beneficios | admissao | demissao
systems_involved: [folha-mensal, integracao-erp] # canônicos do area-mapping.md
symptoms: [valor-de-rubrica-divergente]          # canônicos do symptoms.md
entities: [RUBRICA, MOVIMENTO_FOLHA]             # tabelas/jobs efetivamente tocados
root_cause: |
  Trigger TRG_RUBRICA_VALIDATE estava zerando VLR_RUB quando FLAG_ATIVO mudou
  para 'N' a meio caminho do fechamento, deixando movimentos órfãos.
resolution_query: |
  -- Query que resolveu (a que efetivamente rodou, não a do plano original)
  UPDATE MOVIMENTO_FOLHA
     SET VLR_RUB = (SELECT VLR_BASE FROM RUBRICA WHERE COD_RUB = MOVIMENTO_FOLHA.COD_RUB)
   WHERE COD_RUB = 'INSS01'
     AND DT_REF  = TO_DATE('2026-04-01', 'YYYY-MM-DD')
     AND VLR_RUB = 0;
resolution_steps:
  - "Diagnóstico: SELECT confirmou 47 movimentos com VLR_RUB=0 para a rubrica INSS01."
  - "Reativação manual da rubrica: UPDATE RUBRICA SET FLAG_ATIVO='S' WHERE COD_RUB='INSS01'."
  - "Recálculo dos movimentos via UPDATE acima."
  - "Re-disparo do job CTM JOB_FECHA_FOLHA_MAR2026."
  - "Validação: SELECT confirmou 0 movimentos com VLR_RUB=0."
dossiers_cited:
  - java/folha/schema-rubricas.md
  - controlm/folha/folder-fechamento-mensal.md
reuse_count: 0                                   # incrementado quando outro ticket cita este caso
promoted_to: null                                # path do playbook se promovido (Stage 05)
---

# INC1234567 — Rubrica INSS zerada no fechamento de março

## Contexto

(O que o solicitante reportou; janela; abrangência; criticidade.)

47 funcionários com INSS calculado como zero no fechamento de março/2026. RH detectou ao validar amostra antes do envio à Receita. Precisava ser corrigido antes do prazo de envio (3 dias úteis).

## Investigação

(Como chegou à raiz. Cite os queries de diagnóstico.)

1. SELECT em MOVIMENTO_FOLHA confirmou os 47 casos.
2. Comparação com fechamento de fevereiro: nenhum problema.
3. Análise do log do job de fechamento: warning silencioso de trigger TRG_RUBRICA_VALIDATE.
4. Inspeção da trigger: zerava VLR_RUB se FLAG_ATIVO='N' no momento do INSERT.
5. Histórico do FLAG_ATIVO: alteração feita por DBA durante o fechamento (ticket sem coordenação com RTB).

## Resolução

(Já no frontmatter `resolution_query` + `resolution_steps`. Aqui apenas o que não cabe lá.)

Reativação coordenada com DBA antes do recálculo. Comunicado ao time de Compliance que o número final estava correto.

## Lições

(O que evitaria recorrência. Vira candidato a playbook se virar padrão.)

- Trigger TRG_RUBRICA_VALIDATE deveria escrever em log, não zerar silenciosamente. Abrir story para o time de desenvolvimento.
- Mudanças de FLAG_ATIVO durante janela de fechamento devem passar pelo RTB. Discutir com DBA.
- Diagnóstico via SELECT em MOVIMENTO_FOLHA + JOIN RUBRICA é o primeiro passo padrão para qualquer "rubrica zerada" — promover a playbook se ocorrer ≥3 vezes.
```
