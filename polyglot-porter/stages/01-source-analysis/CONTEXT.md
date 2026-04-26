# Stage 01: Source Analysis

Inventariar o source ICM-compliant. Catalogar módulos, frameworks usados, public surface (classes/funções públicas) e localização do test corpus.

## Inputs

| Fonte | Arquivo / Localização | Escopo | Por quê |
|-------|----------------------|--------|---------|
| Setup | `../../setup/questionnaire.md` | Seção respostas (`source_path`, `source_lang`, `scope`, `test_paths`) | Limites do scan |
| Stage 00 | `../00-preflight/output/preflight-report.md` | Verificar `STATUS: PASS` | Pré-condição |
| Config | `../../_config/language-archetypes.md` | Apenas seção da `source_lang` | Que arquivos/anotações reconhecer |
| Source | `<source_path>` | Conforme `scope` | O alvo |

## Process

1. Confirmar `STATUS: PASS` em preflight-report. Abortar se diferente.
2. Walking do source filtrando por `ignore_paths` e `sensitive_patterns`.
3. Para cada arquivo de código identificar:
   - Linguagem (extensão)
   - Tipo lógico (controller, service, repository, entity, dto, config, test, etc.) usando heurística por anotação/import + naming
   - Public surface (classes/métodos/funções com modificador public ou ausência de prefixo `_` em Python)
   - Frameworks detectados (Spring Boot, ASP.NET, FastAPI, Django, etc.)
4. Construir `surface-map.json` com toda a public surface (chave: FQN; valor: tipo, signature simplificada, file path, linha).
5. Construir `source-inventory.md` com:
   - Resumo: total de arquivos por tipo lógico
   - Frameworks detectados (com versões inferidas do build file)
   - Estrutura de módulos/pacotes
   - Localização do test corpus
   - Build tool e comando de build
6. Detectar dependências (parsing de `pom.xml` / `.csproj` / `pyproject.toml` / `requirements.txt`).
7. Escrever `dependencies.json` com `{name, version, scope}`.

## Checkpoints

Sem checkpoint humano interno. Edição manual de outputs entre stages é permitida (fluxo Unix).

## Audit

| Verificação | Aprovação |
|-------------|-----------|
| Cobertura: cada arquivo do scope no inventário | Sem gaps |
| `surface-map.json` valida JSON | parse OK |
| Frameworks identificados batem com `source_framework` do questionnaire | Sem conflito |
| Test corpus encontrado | Pelo menos 1 path; senão registrar warning |
| Sensitive files: nome registrado, conteúdo NÃO lido | Auditável |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Inventário | `output/source-inventory.md` | MD: resumo + estrutura + frameworks + tests |
| Mapa de superfície | `output/surface-map.json` | JSON: { FQN → {type, signature, file, line} } |
| Dependências | `output/dependencies.json` | JSON: lista de {name, version, scope} |

## Hand-off

Stage 02 lê os 3 outputs + `_config/language-archetypes.md` (target side) para escolher stack-flavor.
