# Playbook — Validação de Logs

Padrões para validar logs em cada stack do RTB-RH. Use no Stage 04 quando o plano envolver "checar log" — sempre indique caminho + padrão + expectativa.

> **Importante**: caminhos abaixo são **convenções comuns**. Confirmar contra o dossiê do sistema concreto ou contra o time de infraestrutura.

---

## Java (aplicações backend)

**Caminho típico**: `/var/log/<sistema>/<sistema>_<YYYY-MM-DD>.log` ou via centralizador (Splunk/ELK).

**Níveis a observar**:
- `ERROR` — falha funcional ou de integração
- `WARN` — degradação ou comportamento inesperado não-fatal
- `INFO` — começo/fim de operações importantes (start de job, end de transação)

**Padrões úteis**:

```bash
# Erros recentes de uma classe específica
grep -E "ERROR.*<NomeClasseSuspeita>" /var/log/<sistema>/<sistema>_<dt>.log | tail -50

# Stack traces de SQLException
grep -B2 -A20 "SQLException" /var/log/<sistema>/<sistema>_<dt>.log

# Linhas em janela temporal
sed -n "/<HH:MM:SS>/,/<HH:MM:SS>/p" /var/log/<sistema>/<sistema>_<dt>.log
```

**O que esperar**:
- Após o fix, **ausência** de novos ERROR/WARN do padrão original.
- Para validações pré-fix, **presença** das mensagens que confirmam a hipótese de root cause.

---

## TIBCO BW

**Caminhos típicos**:
- Logs do Engine: `<TIBCO_HOME>/tra/domain/<domain>/logs/`
- Logs por aplicação: `<TIBCO_HOME>/bw/<release>/logs/<app>/`

**O que checar**:

```bash
# Mensagens de erro do EMS/JMS no engine
grep -E "Error.*EMS|JMS.*Exception" <TIBCO_HOME>/tra/domain/<domain>/logs/bwengine.log

# Falhas em processos específicos
grep "<NomeProcesso>" <TIBCO_HOME>/bw/.../logs/<app>/<app>.log | grep -E "Error|Exception"
```

**Sinais clássicos**:
- `Connection refused` no destino → endpoint downstream fora do ar.
- `Message not delivered` em fila → consumer parado ou erro de schema.
- `Timeout waiting for response` → backend lento ou trava.

**Validação pós-fix**: confirmar que mensagem foi processada (consultar log do consumer downstream OU base de destino).

---

## PowerCenter

**Caminhos típicos**:
- Session log: `<INFA_HOME>/server/infa_shared/SessLogs/<workflow>/<session>.<run>.log`
- Workflow log: `<INFA_HOME>/server/infa_shared/WorkflowLogs/<workflow>.<run>.log`

**O que checar**:

```bash
# Erros de mapping
grep -E "READER_|WRITER_|TRANSF_" <session-log> | grep -E "ERROR|FATAL"

# Linhas rejeitadas (rejection)
grep "Rejected" <session-log>

# Throughput vs. histórico
grep "Total Rows" <session-log>
```

**Sinais clássicos**:
- `RR_4035: SQL Error` → erro na origem (Source Qualifier).
- `WRT_8129: Database errors occurred` → erro no target (constraint, lock).
- `TT_11132` → erro em transformation (lookup not cached, expression).

**Validação pós-fix**: re-rodar com schedule ou via comando `pmcmd startworkflow` e confirmar 0 rejected rows.

---

## Control-M

**Caminhos**:
- Output do job: via console Control-M (View Output) ou diretório `<CTM_OUTPUT_DIR>/<job>.<order_id>`.
- SYSOUT / sysprint dependendo do tipo de job.

**O que checar**:

```bash
# Saída do job
ctmpsm -OUTPUT <job_name> <order_id>

# Estado atual de jobs (ativos, em hold, abortados)
ctmpsm -LISTACT JOBS=ALL ODATE=<YYYYMMDD>
```

**Sinais clássicos**:
- `NOTOK` no status → job abortou. Inspecionar output completo.
- `EXECUTING` por tempo > esperado → job travado. Verificar processo no host.
- `WAITING_HOSTS` → host de execução indisponível.
- `WAITING_QUOTA` → limite de execução simultânea atingido.

**Validação pós-fix**: ordenar/forçar nova execução; confirmar `OK` no status final.

---

## Angular (frontend)

**Onde olhar**:
- Console do navegador (F12 → Console).
- Network tab (F12 → Network) para chamadas HTTP que falharam.
- Logs do servidor backend que serve a API consumida (geralmente Java).

**Sinais clássicos**:
- `Failed to load resource: net::ERR_*` → backend fora do ar ou CORS.
- `5xx` em chamada API → erro no backend (seguir o trace).
- `4xx` (especialmente 401/403) → token expirado ou permissão.
- `0` (sem status) → CORS ou rede.

**Padrão**: tickets de Angular quase sempre exigem inspecionar o **backend Java** correspondente. Use `related:` no dossiê para chegar lá.

---

## Banco Oracle (acesso direto)

**O que checar quando o sintoma é "query lenta" ou "deadlock"**:

```sql
-- Sessions ativas e o que estão executando
SELECT SID, SERIAL#, USERNAME, STATUS, SQL_ID, EVENT, SECONDS_IN_WAIT
  FROM V$SESSION
 WHERE STATUS = 'ACTIVE'
   AND USERNAME IS NOT NULL;

-- Locks e bloqueios
SELECT *
  FROM DBA_BLOCKERS;

-- Top SQLs por tempo
SELECT SQL_ID, ELAPSED_TIME, EXECUTIONS, SQL_TEXT
  FROM V$SQL
 ORDER BY ELAPSED_TIME DESC
 FETCH FIRST 20 ROWS ONLY;
```

**Sinais**:
- `enq: TX - row lock contention` → outro processo segurando lock; investigar quem.
- `library cache lock` → DDL pesado em concorrência.

---

## Padrão geral de validação

Para qualquer fix, o plano de ação deve indicar:

1. **Caminho exato** do log (não "verificar logs").
2. **Padrão a buscar** (regex ou string literal).
3. **Janela temporal** (a partir de quando).
4. **O que esperar** — ausência ou presença, e em qual quantidade.

Exemplo:
> "Verificar `/var/log/folha/folha_2026-04-26.log` por ocorrências de `ERROR.*TRG_RUBRICA_VALIDATE` após 10:00 do dia. Esperado: zero novas ocorrências (já houveram ~50 antes do fix)."
