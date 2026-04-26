# Conventions Reference — Polyglot Porter

Convenções aplicadas ao próprio workspace `polyglot-porter` E aos overlays gerados. Adaptado de `workspace-builder-v2/references/conventions-reference.md`.

## Estrutura de pastas

| Convenção | Aplicação |
|---|---|
| Stage prefix `0N-` | Pastas de pipeline em `stages/` |
| `_` prefix em silos passivos | `_config/` (não tem CONTEXT.md, é só L3) |
| `output/` em cada stage | Local fixo de hand-off |
| `.gitkeep` em pastas vazias | Para garantir presença em git |

## Naming

| Tipo | Convenção |
|---|---|
| Arquivos novos | kebab-case (`pattern-equivalences/java-to-dotnet.md`) |
| Pastas de stage | numeração zero-padded (`00-preflight`, `01-source-analysis`) |
| Schemas | `<entity>.schema.json` |
| Templates | nome do artefato a renderizar (`pom.xml`, `Program.cs.tmpl`) |

No overlay gerado (target):
- Java: pastas `src/main/java/<package>/...`, classes `PascalCase.java`
- .NET: solution + projeto `ProjectName.csproj`, classes `PascalCase.cs`
- Python: pastas snake_case, módulos snake_case, classes PascalCase

## CONTEXT.md de stage

Cada CONTEXT.md tem **apenas** estas seções (na ordem):

1. Título com número e nome do stage
2. Descrição em 1 frase
3. `## Inputs` — tabela
4. `## Process` — passos numerados
5. `## Checkpoints` — opcional
6. `## Audit` — tabela
7. `## Outputs` — tabela
8. `## Hand-off` — 1-2 frases

NÃO incluir: definições extensas, exemplos longos, regras gerais. Esses ficam em `_config/` ou `references/`.

Limite: ≤ 80 linhas idealmente.

## CLAUDE.md raiz

Tem **apenas**:
1. Título
2. Descrição em 2-3 frases (o que o workspace faz, e diferencial)
3. `## Mapa de Pastas` — tree
4. `## Triggers` — tabela
5. `## Roteamento` — tabela
6. `## O Que Carregar` — tabela (selectivity matrix)
7. `## Convenções` — bullets curtos
8. `## Hand-offs entre Stages` — tabela
9. `## Checkpoints Humanos` — tabela

Limite: ≤ 1.000 tokens (~120 linhas).

## CSV (translation-map.csv)

- Encoding UTF-8 sem BOM
- Header obrigatório com nomes exatos das colunas conforme schema
- Separador: vírgula
- Strings com vírgula no conteúdo: aspas duplas
- Validar contra `shared/schemas/translation-map.schema.json` em stage 03 e re-validar em stage 05

## JSON outputs

- 2-space indent
- UTF-8
- Sem comentários (JSON puro)
- Validar contra schema correspondente em `shared/schemas/`

## Referências entre arquivos

- Path relativo no markdown: `[texto](../sibling/file.md)` ou `(../../shared/templates/)`
- Stages referenciam apenas pastas anteriores (DAG). Verificável em audit.

## Layer ratio (do ICM rubric)

| Arquivo | Work context | Behavioral rules | Routing/Map |
|---|---|---|---|
| CLAUDE.md raiz | 20% | 10% | 70% |
| CONTEXT.md raiz | 30% | 0% | 70% |
| stages/<n>/CONTEXT.md | 80% | 20% | 0% |
| `_config/*.md` (L3) | 100% | 0% | 0% |

## Anti-patterns deste workspace

- **CLAUDE.md inflado.** Vira documentação. Limite ≤1.000 tokens.
- **CONTEXT.md poluído.** Inclui regras ou exemplos. Mover para `_config/`.
- **Stages com dependência cruzada.** Stage N só pode ler de stages < N + L3.
- **L3 monolítico.** `_config/` deve ter arquivos focados (1 tópico cada), não mega-arquivos.
- **Skip pattern-equivalences direcional.** Carregar apenas o par ativo (`java-to-dotnet.md`), não todos. Selectivity > caching.
