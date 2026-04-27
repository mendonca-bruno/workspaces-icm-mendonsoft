# rtb-rh — Roteamento de Tarefas

## O Que Este Workspace Faz

Processa tickets de sustentação RTB-RH (folha, ponto, benefícios, admissão, demissão) e mantém uma KB viva destilada de DDLs Oracle, mappings PowerCenter, processos TIBCO BW, jobs Control-M e código Java/Angular. Saída principal de cada ticket: plano de ação com queries SQL específicas + validações de log.

---

## Para Onde Ir

| Você Quer... | Ir Para |
|--------------|---------|
| Ingerir uma DDL/projeto novo na KB | `stages/01-ingest/CONTEXT.md` |
| Começar a tratar um ticket recém-aberto | `stages/02-clarify/CONTEXT.md` |
| Buscar contexto (cases + dossiês) para um ticket já clarificado | `stages/03-search/CONTEXT.md` |
| Montar o plano com queries SQL e validações | `stages/04-plan/CONTEXT.md` |
| Fechar um ticket e arquivar como caso | `stages/05-postmortem/CONTEXT.md` |
| Auditar drift, dossiês desatualizados, padrões a promover | `stages/06-kb-maintenance/CONTEXT.md` |
| Navegar a KB por tecnologia/sistema | `kb/CONTEXT.md` |
| Encontrar um caso resolvido similar | `cases/CONTEXT.md` |
| Copiar template de dossiê / caso / playbook / plano | `reference/` |

**Não leia tudo.** Identifique sua tarefa, carregue apenas o que precisa. Veja a tabela "O Que Carregar" no `CLAUDE.md`.

---

## Fluxo da Pipeline

```
Ticket bruto
    │
    ▼
[02-clarify]  ─── ticket vago? perguntas + edit gate humano ─── loop
    │ (clarificado)
    ▼
[03-search]   ─── cases primeiro → query expansion → roteamento hierárquico em kb/
    │
    ▼
[04-plan]     ─── queries SQL + validações de log + checkpoint humano se destrutivo
    │ (Bruno executa fora do workspace)
    ▼
[05-postmortem] ─── arquiva em cases/ ou propõe playbook se recorrência ≥3
    │
    ▼
(loop) próximo ticket


Pipeline paralelo (não por-ticket):

[01-ingest]         ─── nova fonte → kb/_sources/ + dossiê em kb/<tech>/<sistema>/
[06-kb-maintenance] ─── semanal: drift de hashes, dossiês > 90 dias, clusters não promovidos
```

---

## Recursos Compartilhados

| Recurso | Localização | Quando Carregar |
|---------|-------------|-----------------|
| Taxonomia de sintomas | `kb/_taxonomies/symptoms.md` | Stage 02 (clarificação), Stage 03 (query expansion) |
| Mapa área × sistemas | `kb/_taxonomies/area-mapping.md` | Stage 02, Stage 03 |
| Topic schema por área | `kb/_taxonomies/topic-schema-by-area.md` | Stage 03 |
| Template de dossiê | `reference/dossier-template.md` | Stage 01 |
| Template de caso | `reference/case-template.md` | Stage 05 |
| Template de playbook | `reference/playbook-template.md` | Stage 05 (promoção) |
| Rubrica de clarificação | `reference/clarification-rubric.md` | Stage 02 |
| Template de plano | `reference/plan-template.md` | Stage 04 |
| Patterns SQL por área | `reference/sql-patterns-by-area.md` | Stage 04 |
| Playbook de validação de log | `reference/log-validation-playbook.md` | Stage 04 |
| Template para escrever stage novo | `reference/stage-context-template.md` | Quando criar/refatorar stage |
| Índice agregado de KB | `kb/INDEX.md` | Stage 03 (sempre antes de descer árvore) |
| Índice agregado de cases | `cases/INDEX.md` | Stage 02, Stage 03 (sempre primeiro) |
| Flags de manutenção | `kb/_pending-updates.md` | Stage 06 |
