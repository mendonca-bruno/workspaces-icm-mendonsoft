# Template — Plano de Ação

Use este molde para `tickets/<id>/04-action-plan.md`. Estrutura ordenada para Bruno (ou qualquer membro do time) executar sem ambiguidade.

```markdown
# Plano de Ação — <ticket_id>

## Resumo

(2-3 frases: qual problema, qual estratégia, qual o resultado esperado.)

47 funcionários com INSS calculado como zero. Hipótese: trigger TRG_RUBRICA_VALIDATE zerou os movimentos após mudança de FLAG_ATIVO. Estratégia: confirmar via diagnóstico, reativar rubrica, recalcular movimentos, validar.

## Premissas

(O que precisa ser verdade para o plano funcionar. Se uma falha, voltar para Stage 03.)

- [ ] Rubrica INSS01 está com FLAG_ATIVO='N' indevidamente.
- [ ] DBA confirma que reativação não tem efeito colateral em compliance.
- [ ] Janela de envio à Receita ainda não passou (verificar com Compliance).
- [ ] Job JOB_FECHA_FOLHA_MAR2026 pode ser re-disparado sem coordenação adicional.

## Diagnóstico (queries SELECT — sempre seguros)

> Fonte: kb/java/folha/schema-rubricas.md#fks-e-joins-críticos

```sql
-- 1. Confirmar abrangência
SELECT COUNT(*) AS qtde_zerados, COD_RUB, DT_REF
  FROM MOVIMENTO_FOLHA
 WHERE VLR_RUB = 0
   AND COD_RUB = 'INSS01'
   AND DT_REF  = TO_DATE('2026-04-01', 'YYYY-MM-DD')
 GROUP BY COD_RUB, DT_REF;
-- Esperado: 47

-- 2. Estado atual da rubrica
SELECT COD_RUB, FLAG_ATIVO, DT_ALTERACAO, USR_ALTERACAO
  FROM RUBRICA
 WHERE COD_RUB = 'INSS01';
-- Esperado: FLAG_ATIVO='N' alterada por DBA recentemente
```

## Fix [DESTRUTIVO — checkpoint humano antes de cada UPDATE]

> Fonte: kb/_playbooks/rubrica-zerada-flag-ativo.md (se já existe playbook)

```sql
-- [DESTRUTIVO] 1. Reativar rubrica
UPDATE RUBRICA
   SET FLAG_ATIVO = 'S'
 WHERE COD_RUB = 'INSS01';
-- Esperado: 1 linha afetada

-- [DESTRUTIVO] 2. Recalcular movimentos zerados
UPDATE MOVIMENTO_FOLHA
   SET VLR_RUB = (SELECT VLR_BASE FROM RUBRICA WHERE COD_RUB = MOVIMENTO_FOLHA.COD_RUB)
 WHERE COD_RUB = 'INSS01'
   AND DT_REF  = TO_DATE('2026-04-01', 'YYYY-MM-DD')
   AND VLR_RUB = 0;
-- Esperado: 47 linhas afetadas
```

## Re-execução de jobs (se necessário)

> Fonte: kb/controlm/folha/folder-fechamento-mensal.md

- [ ] Coordenar com plantão CTM para re-disparar `JOB_FECHA_FOLHA_MAR2026`.
- [ ] Verificar dependências downstream: jobs de envio à Receita ainda não devem ter rodado.

## Validação pós-fix

```sql
-- 1. Nenhum movimento zerado deve restar
SELECT COUNT(*)
  FROM MOVIMENTO_FOLHA
 WHERE VLR_RUB = 0
   AND COD_RUB = 'INSS01'
   AND DT_REF  = TO_DATE('2026-04-01', 'YYYY-MM-DD');
-- Esperado: 0

-- 2. Total INSS recolhido deve bater com o esperado
SELECT SUM(VLR_RUB) AS total_inss
  FROM MOVIMENTO_FOLHA
 WHERE COD_RUB = 'INSS01'
   AND DT_REF  = TO_DATE('2026-04-01', 'YYYY-MM-DD');
-- Esperado: <valor de referência fornecido pelo Compliance>
```

## Validação de logs

> Fonte: reference/log-validation-playbook.md#java

- Caminho: `/var/log/folha/job_fechamento_<dt>.log`
- Buscar: `WARN.*TRG_RUBRICA_VALIDATE.*FLAG_ATIVO`
- Esperado: mensagens timestamped do incidente; nenhuma nova após o fix.

## Rollback

(Como reverter cada UPDATE, se algo der errado.)

```sql
-- Rollback do passo Fix.2 (snapshot dos valores anteriores antes de rodar)
-- Antes do UPDATE: SELECT NUM_MATRICULA, COD_RUB, DT_REF, VLR_RUB
--                    FROM MOVIMENTO_FOLHA
--                   WHERE COD_RUB = 'INSS01'
--                     AND DT_REF  = TO_DATE('2026-04-01', 'YYYY-MM-DD');
-- Salvar resultado em tickets/<id>/snapshot-pre-fix.csv
```

## Fontes citadas

(Cada item deste plano que veio de um dossiê/case/playbook deve ser rastreável.)

- `kb/java/folha/schema-rubricas.md` — schema de RUBRICA e MOVIMENTO_FOLHA
- `kb/controlm/folha/folder-fechamento-mensal.md` — coordenação de jobs
- `cases/INC1234567.md` — caso similar resolvido em 2026-03-15
- `kb/_playbooks/rubrica-zerada-flag-ativo.md` — playbook (se aplicável)
```
