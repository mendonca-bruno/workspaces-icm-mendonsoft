# Topic Schema por Área

Vocabulário expandido por área RH: entidades canônicas, sinônimos comuns do solicitante, códigos de erro recorrentes, abreviações internas. Usado pelo Stage 03 para fazer **query expansion** estruturada antes de buscar dossiês.

**Como funciona o expansion**: ticket diz "rubrica do INSS tá zerada" → Stage 03 expande para `[RUBRICA, INSS, valor=0, valor-de-rubrica-divergente OR rubrica-nao-calculou, sistemas: folha-mensal]` antes de filtrar `kb/INDEX.md` e `cases/INDEX.md`.

**Como editar:** quando perceber que o agente não fez o link semântico esperado (ex.: solicitante disse "vale" e agente não pensou em VR), adicione o sinônimo aqui.

---

## Folha

```yaml
entities_canonical:
  RUBRICA:
    aliases: [verba, código de pagamento, código de desconto]
  EVENTO:
    aliases: [movimentação, ocorrência]
  MOVIMENTO_FOLHA:
    aliases: [movto, mov]
  HOLERITE:
    aliases: [contracheque, demonstrativo de pagamento]

terminology:
  inss: [INSS, contribuição previdenciária, instituto]
  irrf: [IRRF, imposto de renda, IR]
  fgts: [FGTS, fundo de garantia]
  liquido: [líquido a receber, valor a pagar]
  bruto: [bruto, salário base, vencimentos]

common_codes:
  rub_inss: <preencher código real>
  rub_irrf: <preencher código real>
  rub_fgts: <preencher código real>
```

## Ponto

```yaml
entities_canonical:
  BATIDA:
    aliases: [marcação, registro de ponto, picada]
  JORNADA:
    aliases: [horário, expediente, escala]
  ESCALA:
    aliases: [turno, regime de trabalho]

terminology:
  hora_extra: [HE, hora extra, sobre-jornada, OT]
  abono: [abono de horas, abonado, justificado]
  falta: [ausência, falta, AT]
  atraso: [atraso, atrasado]
```

## Benefícios

```yaml
entities_canonical:
  BENEFICIO:
    aliases: [vale, auxílio]
  CREDITO:
    aliases: [crédito, recarga, lançamento]

terminology:
  vr: [VR, vale refeição, refeição, auxílio refeição]
  va: [VA, vale alimentação, auxílio alimentação]
  vt: [VT, vale transporte, auxílio transporte]
  pls: [participação nos lucros, PLR]
```

## Admissão

```yaml
entities_canonical:
  CANDIDATO:
    aliases: [candidato, futuro funcionário, novo]
  PROPOSTA:
    aliases: [proposta de emprego, oferta]
  DOCUMENTACAO_ADMISSIONAL:
    aliases: [docs, documentação, papelada]

terminology:
  workflow: [workflow, fluxo, processo, etapa]
  aprovacao_gestor: [aprovação do gestor, OK do gestor]
  exame_admissional: [ASO, exame médico admissional]
```

## Demissão

```yaml
entities_canonical:
  RESCISAO:
    aliases: [rescisão, TRCT, termo de rescisão]
  AVISO_PREVIO:
    aliases: [aviso, AP, aviso prévio trabalhado, AP indenizado]

terminology:
  baixa: [baixa, desligamento, encerramento, demissão]
  homologacao: [homologação, sindicato, assistido]
```

## Transversais (todas as áreas)

```yaml
codes_oracle_common:
  ORA-00001: [unique constraint, chave duplicada]
  ORA-01400: [cannot insert NULL, campo obrigatório]
  ORA-12541: [TNS no listener, banco fora do ar]
  ORA-00942: [table or view does not exist, tabela não existe]

terminology_integracao:
  fila: [fila, queue, JMS, EMS, MQ]
  topico: [tópico, topic]
  consumer: [consumidor, listener, subscriber]
  producer: [produtor, publisher]

terminology_controlm:
  job_aborted: [job abortou, job falhou, NOTOK]
  job_held: [job em hold, segurado, paralisado]
  cyclic: [cíclico, recorrente]
```
