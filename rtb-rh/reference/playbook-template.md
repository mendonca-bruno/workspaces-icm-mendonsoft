# Template — Playbook (padrão promovido de cases recorrentes)

Use este molde ao criar `kb/_playbooks/<sintoma>-<sistema>.md` no Stage 05 (decisão de promoção). Playbook abstrai matrículas/datas/IDs específicos e mantém apenas o padrão de resolução.

```markdown
---
title: <título descritivo>                       # ex.: "Rubrica zerada após mudança de FLAG_ATIVO"
symptoms: [valor-de-rubrica-divergente, rubrica-nao-calculou]   # ids da taxonomia
systems: [folha-mensal]
area: folha
created_from_cases:                              # cases-fonte que originaram este playbook
  - cases/INC1234567.md
  - cases/INC1289012.md
  - cases/INC1300456.md
created: 2026-05-15
last_used: 2026-05-15
use_count: 0                                     # incrementado quando o playbook é citado num plano
related_dossiers:
  - kb/java/folha/schema-rubricas.md
  - kb/controlm/folha/folder-fechamento-mensal.md
---

# <Título>

## Quando aplicar

(Sintomas + sistemas que disparam este playbook. Stage 03 usa para matching.)

- Sintoma: rubrica esperada apareceu zerada (ou não apareceu) após o fechamento.
- Sistema: folha-mensal.
- Sinal adicional típico: trigger ou job upstream alterou estado de RUBRICA durante a janela.

## Diagnóstico (sempre seguro — apenas SELECTs)

```sql
-- 1. Confirmar abrangência
SELECT COUNT(*), COD_RUB, DT_REF
  FROM MOVIMENTO_FOLHA
 WHERE VLR_RUB = 0
   AND COD_RUB = :rubrica_suspeita     -- substituir
   AND DT_REF  = :dt_fechamento        -- substituir
 GROUP BY COD_RUB, DT_REF;

-- 2. Verificar FLAG_ATIVO da rubrica
SELECT COD_RUB, FLAG_ATIVO, DT_ALTERACAO, USR_ALTERACAO
  FROM RUBRICA
 WHERE COD_RUB = :rubrica_suspeita;

-- 3. Inspecionar log do job de fechamento (Control-M)
-- Buscar por: WARN.*TRG_RUBRICA_VALIDATE.*FLAG_ATIVO
```

## Fix padrão

```sql
-- 1. (Se FLAG_ATIVO='N' indevidamente) Reativar a rubrica
UPDATE RUBRICA
   SET FLAG_ATIVO = 'S'
 WHERE COD_RUB = :rubrica_suspeita;

-- 2. Recálcular os movimentos zerados
UPDATE MOVIMENTO_FOLHA
   SET VLR_RUB = (SELECT VLR_BASE FROM RUBRICA WHERE COD_RUB = MOVIMENTO_FOLHA.COD_RUB)
 WHERE COD_RUB = :rubrica_suspeita
   AND DT_REF  = :dt_fechamento
   AND VLR_RUB = 0;

-- 3. Re-disparar job de fechamento se necessário
-- (Coordenar com Control-M: JOB_FECHA_FOLHA_<MES><ANO>)
```

## Validação pós-fix

```sql
-- Confirmar que não restou nenhum movimento zerado
SELECT COUNT(*)
  FROM MOVIMENTO_FOLHA
 WHERE VLR_RUB = 0
   AND COD_RUB = :rubrica_suspeita
   AND DT_REF  = :dt_fechamento;
-- Esperado: 0
```

## Premissas

- Rubrica suspeita é conhecida (caso contrário, fazer triagem por sintomas brutos primeiro).
- Janela de fechamento ainda permite recálculo (não passou do envio à Receita).
- DBA confirma que FLAG_ATIVO pode ser revertido sem efeito colateral.

## Quando NÃO aplicar

- Se FLAG_ATIVO='N' foi intencional (ex.: rubrica deprecada por compliance), não reativar — escalar.
- Se há mais de uma rubrica zerada simultaneamente, suspeitar de causa upstream maior (job de fechamento abortado) — investigar primeiro o folder Control-M.
```
