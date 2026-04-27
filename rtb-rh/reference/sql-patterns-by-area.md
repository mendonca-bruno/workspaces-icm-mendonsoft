# SQL Patterns por Área RH

Padrões idiomáticos de SQL que aparecem repetidamente nos planos de ação do RTB-RH. Use como ponto de partida no Stage 04 — adapte aos nomes reais das tabelas (do dossiê citado).

> **Importante**: nomes de tabelas/colunas abaixo são **placeholders genéricos**. Sempre validar contra o dossiê do sistema concreto. Se o dossiê dá outro nome, o dossiê manda.

---

## Folha

### Diagnóstico de divergência em rubrica

```sql
-- Confirmar abrangência por rubrica e período
SELECT COUNT(*) AS qtde, COD_RUB, DT_REF
  FROM MOVIMENTO_FOLHA
 WHERE <condição>
 GROUP BY COD_RUB, DT_REF;

-- Comparar com período anterior (sanity check)
SELECT COD_RUB, DT_REF, SUM(VLR_RUB) AS total
  FROM MOVIMENTO_FOLHA
 WHERE COD_RUB = :cod
   AND DT_REF IN (:dt_atual, :dt_anterior)
 GROUP BY COD_RUB, DT_REF;
```

### Recalcular movimentos a partir do catálogo

```sql
UPDATE MOVIMENTO_FOLHA
   SET VLR_RUB = (SELECT VLR_BASE FROM RUBRICA WHERE COD_RUB = MOVIMENTO_FOLHA.COD_RUB)
 WHERE <condição-precisa>;
```

### Reverter movimento individual

```sql
UPDATE MOVIMENTO_FOLHA
   SET VLR_RUB = :valor_correto
 WHERE NUM_MATRICULA = :matricula
   AND COD_RUB       = :cod
   AND DT_REF        = :dt_ref;
```

---

## Ponto

### Diagnóstico de batida sumida

```sql
-- Verificar batidas registradas em janela
SELECT NUM_MATRICULA, DT_BATIDA, HR_BATIDA, ORIGEM
  FROM BATIDA
 WHERE NUM_MATRICULA = :matricula
   AND DT_BATIDA BETWEEN :dt_ini AND :dt_fim
 ORDER BY DT_BATIDA, HR_BATIDA;

-- Confirmar se batida chegou no staging mas não foi processada
SELECT *
  FROM STG_BATIDA_INTEGRACAO
 WHERE NUM_MATRICULA = :matricula
   AND DT_BATIDA = :dt
   AND STATUS_PROC = 'PENDENTE';
```

### Inserção manual de batida (após autorização do gestor)

```sql
INSERT INTO BATIDA
  (NUM_MATRICULA, DT_BATIDA, HR_BATIDA, ORIGEM, USR_INCLUSAO, JUSTIFICATIVA)
VALUES
  (:matricula, :dt, :hr, 'MANUAL', :usuario, :justificativa);
```

---

## Benefícios

### Diagnóstico de crédito não disponibilizado

```sql
-- Verificar se crédito foi gerado internamente
SELECT *
  FROM BENEFICIO_CREDITO
 WHERE NUM_MATRICULA = :matricula
   AND TIPO_BENEFICIO = :tipo
   AND DT_REF = :dt;

-- Verificar se foi enviado ao fornecedor
SELECT *
  FROM INTEGRACAO_FORNECEDOR_LOG
 WHERE TIPO_BENEFICIO = :tipo
   AND DT_REF = :dt
   AND STATUS_ENVIO IN ('PENDENTE', 'ERRO');
```

---

## Admissão

### Diagnóstico de workflow travado

```sql
-- Estado atual do workflow
SELECT ID_PROCESSO, ETAPA_ATUAL, STATUS, DT_ULTIMA_ALTERACAO, USR_ALOCADO
  FROM WORKFLOW_ADMISSAO
 WHERE NUM_PROPOSTA = :proposta;

-- Histórico de transições
SELECT ETAPA_ANTERIOR, ETAPA_ATUAL, DT_TRANSICAO, USR_TRANSICAO
  FROM WORKFLOW_ADMISSAO_LOG
 WHERE ID_PROCESSO = :id_proc
 ORDER BY DT_TRANSICAO;

-- Pré-requisitos pendentes (exames, documentos)
SELECT TIPO_PREREQ, STATUS, DT_VENCIMENTO
  FROM ADMISSAO_PREREQUISITO
 WHERE NUM_PROPOSTA = :proposta
   AND STATUS != 'OK';
```

---

## Demissão

### Diagnóstico de baixa não propagada

```sql
-- Estado da rescisão
SELECT *
  FROM RESCISAO
 WHERE NUM_MATRICULA = :matricula;

-- Sistemas downstream que ainda mostram o funcionário ativo
-- (depende dos sistemas — tipicamente: folha, benefícios, portal)
SELECT 'folha' AS sistema, STATUS_FUNC FROM FUNCIONARIO_FOLHA WHERE NUM_MATRICULA = :matricula
UNION ALL
SELECT 'beneficios', STATUS FROM FUNCIONARIO_BENEFICIO WHERE NUM_MATRICULA = :matricula
UNION ALL
SELECT 'portal', ATIVO FROM USUARIO_PORTAL WHERE NUM_MATRICULA = :matricula;
```

---

## Padrões transversais

### Buscar último log de execução de job (Control-M)

```sql
SELECT JOB_NAME, ORDER_DATE, STATUS, START_TIME, END_TIME, OUTPUT
  FROM JOB_AUDIT
 WHERE JOB_NAME = :job
 ORDER BY ORDER_DATE DESC, START_TIME DESC
 FETCH FIRST 5 ROWS ONLY;
```

### Buscar mensagens em fila de erro (TIBCO BW / EMS)

```sql
-- Geralmente o backing store é Oracle ou outra base
SELECT MSG_ID, QUEUE_NAME, ERROR_REASON, DT_INSERCAO, PAYLOAD
  FROM EMS_ERROR_QUEUE
 WHERE QUEUE_NAME LIKE '%RH%'
   AND DT_INSERCAO > :dt_inicio
 ORDER BY DT_INSERCAO DESC;
```

### Auditoria de alterações em tabela crítica

```sql
SELECT TBL_AUDITADA, OPERACAO, DT_OPERACAO, USR_OPERACAO, PK_REGISTRO, VALOR_ANTES, VALOR_DEPOIS
  FROM AUDIT_LOG
 WHERE TBL_AUDITADA = :tabela
   AND PK_REGISTRO  = :pk
 ORDER BY DT_OPERACAO DESC;
```
