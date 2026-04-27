# rtb-rh — Sustentação RTB People & Culture

Workspace ICM do time **RTB (Run the Business) — People & Culture** da F1rst (Santander). Resolve tickets de sustentação dos sistemas de RH (folha, ponto, benefícios, admissão, demissão) e mantém uma base de conhecimento viva que destila DDLs, mappings e fluxos de integração.

**Stack atravessada pelos tickets**: Java, TIBCO BW, PowerCenter, Control-M, Angular, Oracle (DDLs). Maioria das resoluções é via SQL.

## Mapa de Pastas

```
rtb-rh/
├── CLAUDE.md                       (você está aqui)
├── CONTEXT.md                      (roteador de tarefas)
├── PROGRESS.md                     (estado vivo, métricas)
├── DECISIONS.md                    (ADRs)
│
├── kb/                             base de conhecimento
│   ├── CONTEXT.md                  roteador "qual tecnologia/sistema?"
│   ├── INDEX.md                    tabela auto-gerada (frontmatter agregado)
│   ├── _sources/                   L4 — DDLs, XMLs BW, mappings PWC, JILs originais
│   ├── _playbooks/                 L3 — promovidos de cases recorrentes
│   ├── _taxonomies/                vocabulário canônico (sintomas, áreas, entidades)
│   ├── _pending-updates.md         flags abertas pelo stage 05 para o stage 06
│   ├── java/<sistema>/<dossie>.md
│   ├── tibco/<sistema>/<dossie>.md
│   ├── powercenter/<sistema>/<dossie>.md
│   ├── controlm/<sistema>/<dossie>.md
│   └── angular/<sistema>/<dossie>.md
│
├── cases/                          tickets resolvidos tipados
│   ├── CONTEXT.md
│   ├── INDEX.md                    symptom × ticket_id × reuse_count
│   └── INC*.md                     1 caso = 1 arquivo, frontmatter obrigatório
│
├── tickets/                        L4 working — execução em andamento
│   └── <ticket-id>/
│       ├── input.md
│       ├── 02-clarification.md
│       ├── 02-clarified.md
│       ├── 03-search-result.md
│       ├── 04-action-plan.md
│       └── 05-postmortem.md
│
├── stages/                         pipeline operacional
│   ├── 01-ingest/CONTEXT.md
│   ├── 02-clarify/CONTEXT.md       (U-shape, human edit gate)
│   ├── 03-search/CONTEXT.md        (cases primeiro, KB hierárquica depois)
│   ├── 04-plan/CONTEXT.md
│   ├── 05-postmortem/CONTEXT.md    (promove padrões a playbooks)
│   └── 06-kb-maintenance/CONTEXT.md (offline semanal)
│
└── reference/                      L3 — templates e rubricas estáveis
    ├── dossier-template.md
    ├── case-template.md
    ├── playbook-template.md
    ├── clarification-rubric.md
    ├── plan-template.md
    ├── sql-patterns-by-area.md
    ├── log-validation-playbook.md
    └── stage-context-template.md
```

## Triggers

| Keyword | Ação |
|---------|------|
| `ingerir <fonte>` | Stage 01 — parse, classifica, propõe destino na KB |
| `processar <ticket-id>` | Pipeline `02 → 03 → 04` para o ticket |
| `clarificar <ticket-id>` | Forçar entrada em Stage 02 |
| `pos-mortem <ticket-id>` | Stage 05 — arquiva caso ou promove a playbook |
| `manutencao` | Stage 06 — drift de hashes, dossiês desatualizados |
| `status` | Tickets em andamento + cases por área + dossiês por área + playbooks ativos |

### Como `status` funciona

Escanear `tickets/*/` (in-flight), `cases/*.md` (resolvidos), `kb/*/*/*.md` (dossiês), `kb/_playbooks/*.md` (playbooks). Renderizar:

```
Status: rtb-rh

Tickets in-flight: N
  └─ por stage atual: 02-clarify=X, 03-search=Y, 04-plan=Z

Cases por área:    folha=N, ponto=N, beneficios=N, admissao=N, demissao=N
Dossiês por tech:  java=N, tibco=N, powercenter=N, controlm=N, angular=N
Playbooks ativos:  N
Pending-updates:   N flags abertas
```

## Roteamento

| Tarefa | Ir Para |
|--------|---------|
| Ingerir nova fonte (DDL, BW, PWC, JIL, código) | `stages/01-ingest/CONTEXT.md` |
| Clarificar ticket vago | `stages/02-clarify/CONTEXT.md` |
| Buscar contexto para ticket | `stages/03-search/CONTEXT.md` |
| Gerar plano de ação | `stages/04-plan/CONTEXT.md` |
| Arquivar caso resolvido | `stages/05-postmortem/CONTEXT.md` |
| Manutenção semanal de KB | `stages/06-kb-maintenance/CONTEXT.md` |
| Buscar conhecimento por tecnologia/sistema | `kb/CONTEXT.md` |
| Buscar caso resolvido similar | `cases/CONTEXT.md` |

## O Que Carregar

| Tarefa | Carregar | NÃO Carregar |
|--------|----------|--------------|
| Ingerir fonte | `kb/INDEX.md`, `kb/<tech>/CONTEXT.md`, `reference/dossier-template.md` | Outros stages, `cases/`, `tickets/` |
| Clarificar | `tickets/<id>/input.md`, `kb/_taxonomies/symptoms.md`, `cases/INDEX.md`, `reference/clarification-rubric.md` | Dossiês de KB, `kb/_sources/` |
| Buscar | `tickets/<id>/02-clarified.md`, `cases/INDEX.md`, `kb/INDEX.md`, `kb/<tech>/CONTEXT.md` das tecnologias suspeitas | Stages 04-06, `kb/_sources/` |
| Plano | `tickets/<id>/02-clarified.md`, `tickets/<id>/03-search-result.md`, dossiês e playbooks citados, `reference/plan-template.md`, `reference/sql-patterns-by-area.md` | Outros tickets, `kb/_sources/`, taxonomias |
| Pós-mortem | `tickets/<id>/04-action-plan.md`, outcome do Bruno, `cases/INDEX.md`, `reference/case-template.md` | Outros tickets, KB de outras áreas |
| Manutenção | Hashes em `kb/_sources/`, frontmatters dos dossiês, `cases/INDEX.md`, `kb/_pending-updates.md` | Stages 01-05, `tickets/` |

## Princípios invioláveis

1. **Cases primeiro**: Stage 03 sempre consulta `cases/INDEX.md` antes de qualquer dossiê de KB.
2. **Sources são canônicos**: arquivos em `kb/_sources/` (L4) são fonte da verdade para campos exatos. Dossiês em `kb/<tech>/<sistema>/*.md` (L3) são destilação semântica reusável. Frontmatter do dossiê tem `source + source_sha256 + last_distilled` que o Stage 06 audita.
3. **Tickets vagos param**: Stage 02 tem human edit gate obrigatório; pipeline não avança sem score mínimo de clarificação.
4. **Recorrência vira playbook**: ≥3 cases com mesmos `symptoms+systems_involved` disparam proposta de promoção a `kb/_playbooks/` no Stage 05.
5. **Não duplique sources em dossiês**: o dossiê referencia (`source: ../_sources/...`), não copia o DDL inteiro.

## Stage Handoffs

Cada stage escreve em `tickets/<id>/<NN>-<nome>.md`. O próximo stage lê de lá. Editar um output entre stages é esperado e absorvido pelo próximo (file-as-edit-surface — princípio ICM).

Stages que não são por-ticket:
- `01-ingest` opera sobre `kb/_inbox/` ou source colado, escreve em `kb/`.
- `06-kb-maintenance` opera sobre toda a KB, escreve `kb-health-report.md` na raiz.
