# Taxonomia de sintomas (canônica)

Vocabulário controlado de sintomas de tickets RTB-RH. Stage 02 usa para normalizar a frase do solicitante (ex.: "salário tá errado") em um sintoma canônico (ex.: `valor-de-rubrica-divergente`). Stage 03 usa para query expansion ao buscar em `cases/INDEX.md` e `kb/INDEX.md`.

**Como editar:** quando um ticket trouxer um sintoma novo (não mapeado), adicione aqui no Stage 02. Não use sinônimos vagos — escolha um nome canônico e liste variantes em `aliases`.

## Schema

```yaml
- id: <slug-curto>
  area: <folha|ponto|beneficios|admissao|demissao|transversal>
  systems_typical: [<sistema>]
  entities_typical: [<TABELA|JOB|MAPPING>]
  aliases: [<frase comum do solicitante>, <variante>]
  description: <1 linha>
```

## Sintomas (seed — preencher conforme uso real)

### Folha

```yaml
- id: valor-de-rubrica-divergente
  area: folha
  systems_typical: [folha-mensal]
  entities_typical: [RUBRICA, EVENTO, MOVIMENTO_FOLHA]
  aliases:
    - "salário errado"
    - "rubrica calculou errado"
    - "valor diferente do esperado em holerite"
    - "divergência em folha"
  description: Valor calculado de uma rubrica não bate com o esperado.

- id: rubrica-nao-calculou
  area: folha
  systems_typical: [folha-mensal, folha-13]
  entities_typical: [RUBRICA, EVENTO]
  aliases:
    - "rubrica zerada"
    - "não veio no holerite"
    - "rubrica faltando"
  description: Rubrica esperada não foi gerada para o funcionário no fechamento.

- id: holerite-nao-disponivel
  area: folha
  systems_typical: [portal-rh, folha-mensal]
  aliases:
    - "não consigo ver meu contracheque"
    - "holerite não aparece"
  description: Holerite gerado mas não exibido no portal do funcionário.
```

### Ponto

```yaml
- id: ponto-nao-bate
  area: ponto
  systems_typical: [ponto-eletronico]
  entities_typical: [BATIDA, JORNADA, ESCALA]
  aliases:
    - "horas erradas"
    - "ponto divergente"
    - "horas extras não calculadas"
  description: Cálculo de jornada/horas extras diverge do esperado.

- id: batida-nao-registrada
  area: ponto
  systems_typical: [ponto-eletronico, integracao-catraca]
  entities_typical: [BATIDA]
  aliases:
    - "ponto sumiu"
    - "não bateu"
    - "ponto não foi para o sistema"
  description: Batida feita pelo funcionário não chegou ao sistema central.
```

### Benefícios

```yaml
- id: beneficio-nao-disponibilizado
  area: beneficios
  systems_typical: [beneficios-portal, integracao-fornecedor]
  aliases:
    - "vale refeição não veio"
    - "benefício não creditado"
    - "cartão sem saldo"
  description: Benefício foi processado internamente mas não chegou ao funcionário/fornecedor.
```

### Admissão / Demissão

```yaml
- id: admissao-presa-em-workflow
  area: admissao
  systems_typical: [workflow-admissao, integracao-erp]
  aliases:
    - "admissão travada"
    - "funcionário não foi admitido"
    - "processo de admissão parado"
  description: Workflow de admissão não avança de uma etapa para a próxima.

- id: demissao-nao-baixou
  area: demissao
  systems_typical: [workflow-demissao, folha-mensal, beneficios-portal]
  aliases:
    - "funcionário desligado ainda aparece ativo"
    - "demissão não processou"
    - "não consegui dar baixa"
  description: Funcionário desligado permanece ativo em sistemas downstream após processamento da demissão.
```

### Transversais

```yaml
- id: job-nao-rodou
  area: transversal
  systems_typical: [controlm]
  entities_typical: [JOB, FOLDER]
  aliases:
    - "job falhou"
    - "rotina não executou"
    - "fluxo parou"
  description: Job orquestrado pelo Control-M não executou ou abortou.

- id: integracao-nao-passou
  area: transversal
  systems_typical: [tibco-bw, powercenter]
  aliases:
    - "dados não chegaram"
    - "integração travada"
    - "fluxo de dados parou"
  description: Fluxo de dados entre dois sistemas não completou.
```
