# Exemplo de Workspace: API Fullstack

Resumo de um workspace para desenvolvimento de API fullstack. Use como referência ao construir novos workspaces de desenvolvimento para entender como um workspace dev ICM completo se parece.

---

## Visão Geral

**O que faz:** Gerencia o ciclo completo de desenvolvimento de uma API backend com painel admin e app mobile/PWA.
**Stages:** 5 (planejamento, contratos, implementação, testes, deploy)
**Usuário alvo:** Desenvolvedores backend/fullstack trabalhando em APIs REST ou GraphQL
**Tech stack exemplo:** Python + FastAPI + PostgreSQL + React PWA

---

## Resumo dos Stages

### 01-planning: Requisitos para Decisões de Arquitetura
- **Input:** Ticket, feature request, ou bug report do usuário
- **Referências carregadas:** Documentação de arquitetura, convenções de código, decisões anteriores (DECISIONS.md)
- **Checkpoints:**
  - Após análise de requisitos: agente apresenta escopo, impacto em módulos existentes, e perguntas de clarificação
  - Após decisão de arquitetura: agente apresenta abordagem proposta, humano valida
- **Audit:** Escopo definido, impacto mapeado, approach escolhido tem justificativa
- **Output:** Documento de planejamento com escopo, approach, e critérios de aceitação
- **Superfície de edição:** O documento de planejamento. Usuário pode refinar escopo e approach.

### 02-contracts: Planejamento para Especificações Técnicas
- **Input:** Documento de planejamento do 01-planning/output/
- **Referências carregadas:** Schemas existentes em `shared/schemas/`, API specs em `shared/types/`
- **Verify:** Schemas validam contra Pydantic/TypeScript compiler; novos endpoints seguem naming conventions
- **Audit:** Contratos completos (request/response/errors), backward compatibility verificada, migrations planejadas
- **Output:** Schemas atualizados, API spec, plano de migration
- **Princípio-chave:** O contrato define O QUÊ a API faz. A implementação decide COMO.

### 03-implementation: Contratos para Código
- **Input:** Schemas e specs do 02-contracts/output/
- **Referências carregadas:** Convenções de código, patterns de implementação, component registry
- **Verify:** `ruff check src/`, `mypy src/`, `pytest --co` (coleta sem executar)
- **Audit:** Segue patterns do codebase, type hints completas, docstrings em portugues, error handling
- **Output:** Código implementado (routes, services, models, schemas)
- **Princípio-chave:** O implementador tem liberdade criativa dentro dos contratos (O QUÊ) e convenções (qualidade).

### 04-testing: Implementação para Testes Verificados
- **Input:** Código do 03-implementation/output/
- **Referências carregadas:** Guia de testes, fixtures existentes, conftest.py
- **Verify:** `pytest -v --cov=src --cov-fail-under=80`
- **Audit:** Testes cobrem happy path + edge cases + error cases, fixtures são reutilizáveis, nomes são descritivos
- **Output:** Testes unitários e de integração passando
- **Nota:** Cobertura mínima é quality gate. Abaixo de 80% o stage não avança.

### 05-deployment: Testes para Deploy Verificado
- **Input:** Código testado do 04-testing/
- **Referências carregadas:** Configuração de infra, Docker configs, variáveis de ambiente
- **Verify:** `docker build .`, `alembic upgrade head --sql` (dry run), CI pipeline passa
- **Audit:** Migration é reversível, env vars documentadas, rollback plan definido
- **Output:** Código deployado ou pronto para deploy
- **Nota:** Este stage é opcional para desenvolvimento local. Obrigatório para produção.

---

## Contexto Compartilhado

### Shared Directory (shared/)
```
shared/
├── types/            ← TypeScript interfaces (PWA ↔ API contract)
│   ├── order.ts
│   └── driver.ts
├── schemas/          ← Pydantic models (API ↔ Database contract)
│   ├── order.py
│   └── driver.py
└── constants/        ← Valores compartilhados
    └── config.py
```

Os schemas em `shared/` são a fonte canônica. Stages referenciam, nunca duplicam.

### Configuração (_config/)
```
_config/
├── tech-stack.md     ← Python 3.12, FastAPI, PostgreSQL, React, etc.
└── environments.md   ← Dev, staging, prod + variáveis por ambiente
```

---

## Scripts de Automação (_scripts/)

```
_scripts/
├── verify-contracts.sh    ← Valida schemas contra compiler
├── verify-implementation.sh ← Lint + type-check + test collection
├── verify-tests.sh        ← Roda testes com cobertura mínima
├── verify-deployment.sh   ← Build Docker + migration dry-run
└── run-all-checks.sh     ← Executa tudo em sequência
```

Cada script corresponde a uma seção Verify em um stage CONTEXT.md.

---

## State Tracking

### PROGRESS.md (Root)
```markdown
# Progress

## Fase Atual
Stage 03 — Implementação do endpoint de otimização de rotas

## Última Sessão
- **Data:** 2026-04-25
- **Realizado:** Implementado GET /routes/optimized com OR-Tools VRP
- **Decisões:** Usar cache de 5min para resultados de otimização

## Próximos Passos
1. Escrever testes para o optimizer service
2. Adicionar tratamento de timeout para otimizações longas
3. Documentar o endpoint no OpenAPI spec

## Padrões de Edição Recorrentes
| Stage | O Que Você Edita | Sugestão de Fix no Source |
|-------|-----------------|--------------------------|
| 03-implementation | Sempre adiciono try/except em services | Adicionar error handling template no reference |
```

### DECISIONS.md (docs/)
Registro de decisões arquiteturais usando ADR simplificado.

---

## Estratégia de Placeholders

**O que tem placeholders:** Nome do projeto, tech stack (linguagem, framework, banco), convenções de naming, comandos de dev, comandos de teste, estrutura de deploy, variáveis de ambiente, schemas core.

**O que é hardcoded:** Estrutura de pastas, fluxo de stages, patterns de verificação, estrutura de templates, processo de audit, tabelas de checkpoint.

**Regra:** Se varia entre projetos, é placeholder. Se é parte da estrutura do framework, é hardcoded.

---

## Onboarding

12 perguntas flat respondidas de uma vez:
- Nome do projeto, linguagens, frameworks, banco de dados, package manager, test runner, estratégia de branching, CI platform, convenções de naming, comandos de dev/test/build, variáveis de ambiente, deploy target

Dados detectados na análise (stage 01) aparecem como defaults. Usuário confirma ou ajusta.
