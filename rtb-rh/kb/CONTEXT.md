# kb/ — Roteamento da Base de Conhecimento

Roteador L1 da KB. Use este arquivo para descobrir qual tecnologia/sistema tem o dossiê relevante para a sua tarefa, sem precisar varrer todos os `.md`.

## Como navegar

1. Identifique a **tecnologia** envolvida (Java? TIBCO BW? PowerCenter? Control-M? Angular?). Se múltiplas, comece pela mais provável.
2. Vá ao `CONTEXT.md` da tecnologia: `kb/<tech>/CONTEXT.md`. Lá você encontra os sistemas cobertos.
3. Vá ao `CONTEXT.md` do sistema: `kb/<tech>/<sistema>/CONTEXT.md`. Lá lista os dossiês.
4. Carregue 1-3 dossiês candidatos (não mais).
5. Siga `related:` no frontmatter para expandir o set se preciso.

**Atalho:** Para tarefas amplas (ex.: "preciso entender folha"), use `kb/INDEX.md` primeiro — tabela agregada de todos os dossiês com sintomas, entidades e queries típicas.

## Tecnologias cobertas

| Tecnologia | Roteador | Quando usar |
|------------|----------|-------------|
| Java | `java/CONTEXT.md` | Aplicações backend, APIs REST, regras de negócio |
| TIBCO BW | `tibco/CONTEXT.md` | Integrações entre sistemas, mensageria |
| PowerCenter | `powercenter/CONTEXT.md` | ETL, cargas batch entre bases |
| Control-M | `controlm/CONTEXT.md` | Orquestração de jobs, dependências de execução |
| Angular | `angular/CONTEXT.md` | Frontends de portais e ferramentas internas |

## Recursos transversais

| Recurso | Onde | Para quê |
|---------|------|----------|
| INDEX agregado | `INDEX.md` | Visão de todos os dossiês de todas as tecnologias |
| Sources canônicos | `_sources/<tech>/<sistema>/` | DDLs, XMLs, JILs, mappings originais (L4) |
| Playbooks | `_playbooks/<sintoma>.md` | Padrões promovidos de cases recorrentes |
| Taxonomias | `_taxonomies/` | Vocabulário canônico (sintomas, áreas, entidades) |
| Pending updates | `_pending-updates.md` | Flags abertas pelo Stage 05 para Stage 06 reconciliar |

## Áreas RH (transversais a todas as tecnologias)

Cada dossiê declara `area:` no frontmatter. Áreas válidas: `folha`, `ponto`, `beneficios`, `admissao`, `demissao`. Mapeamento área × sistemas em `_taxonomies/area-mapping.md`.

Para buscar "tudo de folha", use o `INDEX.md` filtrando por `area: folha`.
