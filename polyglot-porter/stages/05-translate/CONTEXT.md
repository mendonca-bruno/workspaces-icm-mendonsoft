# Stage 05: Translate

Traduzir código + testes file-by-file conforme `translation-map.csv`. O grosso do trabalho LLM mora aqui.

## Inputs

| Fonte | Arquivo / Localização | Escopo | Por quê |
|-------|----------------------|--------|---------|
| Stage 03 | `../03-pattern-mapping/output/translation-map.csv` | Completo (autoridade) | Contrato de tradução por arquivo |
| Stage 02 | `../02-target-discovery/output/reference-distillation.md` | Completo (se existir) | Naming/error-style/DI patterns das references — oracle de estilo |
| Stage 04 | `<overlay>/` | Estrutura do scaffold | Onde escrever |
| Source | `<source_path>` | Conforme CSV | Código a traduzir |
| Config | `../../_config/pattern-equivalences/<source>-to-<target>.md` | Completo | Tabela durante tradução |
| Config | `../../_config/idiom-mapping/*` | Apenas seções relevantes ao arquivo atual | Decisões de idioma |
| References | `../../references/golden-examples/` (apenas o arquivo do target: `spring-petclinic.md` se java, `eshop-on-web.md` se dotnet, `full-stack-fastapi-template.md` se python) | Seção "Expected shape" + "Componentes representativos" | Oracle de shape |

## Process

Para cada linha do CSV com `action != skip`:

1. Ler arquivo source.
2. Aplicar `pattern-equivalences` para mapear anotações/atributos/imports.
3. Aplicar `idiom_decision` da linha CSV (literal vs refactor-to-target-idiom).
4. Aplicar `null_strategy`, `error_strategy`, `concurrency_strategy` conforme CSV.
5. Aplicar naming canônico. **Precedência:** se `reference-distillation.md` define padrões (ex: prefixo `I` em interfaces dotnet, sufixo `Async` em métodos async, namespace por feature em vez de por camada), usar. Senão, default da linguagem (PascalCase / snake_case).
6. Aplicar `transactional` flag se presente (envolver método com `@Transactional` / `BeginTransaction` / `transaction.atomic`).
7. Escrever arquivo target em `target_path` (resolvido relativo ao overlay).
8. Registrar em `translation-log.md`: source → target, idiom_decision aplicada, ajustes manuais (se LLM julgou necessário).
9. Para arquivos de teste: traduzir para framework do target (xUnit / pytest / JUnit) conforme CSV `target_type=test`.

Estratégia de batch: traduzir em ordem topológica (entities → repositories → services → controllers) para minimizar referências a código ainda não traduzido.

Re-runs granulares: `/translate-file --source=<path>` permite re-traduzir 1 arquivo sem rodar pipeline inteiro.

## Checkpoints

Sem gate humano formal interno. Audit por arquivo gera lista de itens com low-confidence ou desvio do CSV.

## Audit

| Verificação | Aprovação |
|-------------|-----------|
| Cada linha CSV com `action != skip` tem arquivo target gerado | Sem orphans |
| Imports/dependencies do target referenciam classes/módulos existentes | Sem broken refs |
| Naming convention aplicada | PascalCase em C#/Java, snake_case em Python |
| Anti-pattern audit por arquivo | Sem catch-and-swallow, sem bare except, sem GIL-on-cpu-bound |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Código traduzido | `<overlay>/` | Source tree do target |
| Log de tradução | `output/translation-log.md` | MD: entrada por arquivo, decisões aplicadas, alertas |

## Hand-off

Stage 06 lê o overlay completo + corpus de testes do source + `equivalence-rubric.md` para avaliar.
