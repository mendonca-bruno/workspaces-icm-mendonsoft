# Exemplo de Workspace: Pipeline de Dados Python

Resumo de um workspace para pipeline de processamento de dados. Use como referência ao construir workspaces para ETL, análise de dados, ou automação de relatórios.

---

## Visão Geral

**O que faz:** Processa dados brutos de múltiplas fontes, transforma, analisa e gera relatórios automatizados.
**Stages:** 4 (ingestão, transformação, análise, relatórios)
**Usuário alvo:** Analistas de dados e engenheiros de dados
**Tech stack exemplo:** Python + pandas + SQLAlchemy + PostgreSQL

---

## Resumo dos Stages

### 01-ingestion: Fontes para Dados Brutos
- **Input:** Configuração de fontes (APIs, CSVs, banco de dados)
- **Referências carregadas:** Documentação das APIs, schemas de dados, credenciais (via env vars)
- **Verify:** Dados extraídos existem no formato esperado, contagem de registros bate com fonte
- **Audit:** Todas as fontes configuradas extraem com sucesso, dados brutos são salvos com timestamp
- **Output:** Dados brutos em `output/raw/` com metadados de extração
- **Nível de automação:** Totalmente automático (scheduled job)

### 02-transformation: Dados Brutos para Dados Limpos
- **Input:** Dados brutos do 01-ingestion/output/
- **Referências carregadas:** Regras de limpeza, schemas de validação, business rules
- **Verify:** `python -m pytest tests/test_transformations.py`, schema validation passa
- **Audit:** Sem nulls em campos obrigatórios, tipos corretos, duplicatas removidas
- **Output:** Dados limpos em `output/clean/` com log de transformações
- **Nível de automação:** Totalmente automático com alertas em anomalias

### 03-analysis: Dados Limpos para Insights
- **Input:** Dados limpos do 02-transformation/output/
- **Referências carregadas:** Métricas de negócio, templates de análise, dados históricos
- **Checkpoints:**
  - Após análise exploratória: agente apresenta achados preliminares, humano decide direção
- **Audit:** Métricas calculadas corretamente, comparação com período anterior, outliers explicados
- **Output:** Resultados de análise em `output/insights/` com visualizações
- **Nível de automação:** Review gate (humano valida insights antes de prosseguir)

### 04-reporting: Insights para Relatórios
- **Input:** Insights do 03-analysis/output/
- **Referências carregadas:** Templates de relatório, regras de formatação, destinatários
- **Verify:** Relatório gera sem erros, totais batem com análise
- **Audit:** Formatação correta, linguagem clara, gráficos legíveis
- **Output:** Relatórios em `output/reports/` (PDF, CSV, ou dashboard)
- **Nível de automação:** Totalmente automático (geração) + review gate (envio)

---

## Contexto Compartilhado

### Shared Directory (shared/)
```
shared/
├── schemas/           ← Pydantic models para validação de dados
│   ├── sales.py       ← Schema de vendas
│   └── payments.py    ← Schema de pagamentos
├── utils/             ← Funções compartilhadas
│   ├── date_utils.py  ← Parsing e formatação de datas
│   └── money_utils.py ← Formatação monetária
└── constants/
    └── business_rules.py ← Regras de negócio (taxas, limites)
```

### Configuração (_config/)
```
_config/
├── data-sources.md    ← URLs, credenciais (ref a env vars), formatos
└── output-formats.md  ← PDF, CSV, Dashboard — formato por tipo de relatório
```

---

## Scripts de Automação (_scripts/)

```
_scripts/
├── verify-schemas.py      ← Valida dados contra schemas Pydantic
├── run-pipeline.py        ← Executa pipeline completo (01→04)
├── run-tests.py           ← Roda todos os testes
└── run-all-checks.sh      ← Lint + tests + schema validation
```

---

## Estratégia de Placeholders

**O que tem placeholders:** Nome do projeto, fontes de dados (URLs, banco, paths), schemas de dados, regras de negócio, métricas de análise, templates de relatório, destinatários, schedule de execução.

**O que é hardcoded:** Estrutura de pipeline (4 stages), fluxo de dados, patterns de verificação, estrutura de scripts.

---

## Onboarding

10 perguntas flat:
- Nome do pipeline, fontes de dados (APIs, CSVs, bancos), formato de output (PDF, CSV, dashboard), métricas de negócio, schedule (diário, semanal, mensal), banco de dados, regras de limpeza, destinatários de relatórios, variáveis de ambiente, tool preferences (pandas vs polars)
