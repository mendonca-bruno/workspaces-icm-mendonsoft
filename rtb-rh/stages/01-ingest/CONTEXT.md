# Stage 01: Ingestão de Fonte

Parsear uma fonte nova (DDL Oracle, processo TIBCO BW, mapping PowerCenter, JIL Control-M, classe Java, módulo Angular) e produzir/atualizar um dossiê semântico em `kb/<tech>/<sistema>/`.

## Inputs

| Fonte | Arquivo/Localização | Escopo | Por quê |
|-------|---------------------|--------|---------|
| Source bruto | Caminho colado pelo Bruno (ou `kb/_inbox/<arquivo>`) | Arquivo completo | A fonte a destilar |
| Índice agregado | `../../kb/INDEX.md` | Tabela completa | Detectar dossiê existente para merge vs. novo |
| Roteador da tecnologia | `../../kb/<tech>/CONTEXT.md` | Lista de sistemas | Conhecer arquétipos existentes |
| Template de dossiê | `../../reference/dossier-template.md` | Arquivo completo | Schema obrigatório |
| Mapa área × sistemas | `../../kb/_taxonomies/area-mapping.md` | Tabela da área provável | Validar nome canônico de sistema |

## Processo

1. Detectar **tipo da fonte** por extensão e conteúdo: DDL (.sql), BW (.process/.bwm/.xml), PWC (.XML), Control-M (.jil), Java (.java/repo), Angular (.ts/.html). Ambíguo → perguntar ao Bruno.
2. Detectar **tecnologia destino**: java | tibco | powercenter | controlm | angular.
3. Detectar **sistema destino**: extrair do nome do arquivo, primeiras linhas, ou metadados; conferir contra `kb/_taxonomies/area-mapping.md`. Se não bate, propor sistema novo.
4. Detectar **área RH**: folha | ponto | beneficios | admissao | demissao (do `area-mapping.md`).
5. Extrair **entidades**: tabelas/jobs/mappings/endpoints presentes na fonte.
6. Decidir **destino**: dossiê novo (`kb/<tech>/<sistema>/<dossie-novo>.md`) ou merge em dossiê existente (verificar `INDEX.md` por overlap de entidades).
7. **[Checkpoint]** — Apresentar ao Bruno: tipo detectado, tecnologia, sistema, área, entidades extraídas, destino proposto (novo ou merge). Bruno aprova ou edita.
8. **Gravar source canônico**: copiar a fonte para `../../kb/_sources/<tech>/<sistema>/<arquivo-original>`. Calcular SHA-256.
9. **Destilar conteúdo** seguindo `reference/dossier-template.md`:
   - Tabelas e relacionamentos (FKs, índices críticos)
   - Joins críticos (combinações usadas em queries reais)
   - Regras de negócio inferidas (CHECK constraints, triggers, código)
   - Padrões de uso (queries típicas em produção que tocam essas entidades)
   - Pointer ao source: `source: ../_sources/<tech>/<sistema>/<arquivo>` + `source_sha256: <hash>` + `last_distilled: <hoje>`
10. **Caso novo**: criar `kb/<tech>/<sistema>/<dossie>.md` + atualizar `kb/<tech>/<sistema>/CONTEXT.md` (se sistema novo, criar a pasta e o CONTEXT também).
11. **Caso merge**: 3-way merge semântico com dossiê existente — preservar `## Casos referenciados`, atualizar `## Tabelas`, adicionar nova seção `## Histórico` se houve mudança não-trivial. **[Checkpoint]** — Bruno resolve contradições.
12. Atualizar `kb/INDEX.md` com a nova entrada (ou rodar Stage 06 depois para regenerar).

## Checkpoints

| Após Passo | Agente Apresenta | Humano Decide |
|------------|-------------------|---------------|
| 7 | Tipo, tecnologia, sistema, área, entidades, destino proposto | Aprovar ou editar destino |
| 11 | Diff de merge proposto vs. dossiê existente; contradições destacadas | Como resolver cada contradição |

## Verify

| Verificação | Comando | Condição de Aprovação |
|-------------|---------|----------------------|
| Hash do source persistido | `sha256sum kb/_sources/<tech>/<sistema>/<arquivo>` | Igual ao registrado no frontmatter |
| Frontmatter completo | Inspecionar dossiê | Campos obrigatórios preenchidos: title, area, systems, entities, source, source_sha256, last_distilled |
| INDEX atualizado | grep do dossiê em `kb/INDEX.md` | Linha presente |

## Audit

| Verificação | Condição de Aprovação |
|-------------|----------------------|
| Dossiê coeso | Tamanho ≤ ~400 linhas; se ultrapassar, considerar split |
| Sem duplicação de source | Conteúdo do dossiê não copia o DDL/XML inteiro — apenas referencia |
| `related:` válido | Todo path em `related:` aponta para arquivo existente |
| Sistema canônico | `systems:` usa nomes do `area-mapping.md`, não nomes livres |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Source canônico | `../../kb/_sources/<tech>/<sistema>/<arquivo>` | Original (SQL/XML/JIL/etc.) |
| Dossiê (novo ou atualizado) | `../../kb/<tech>/<sistema>/<dossie>.md` | Markdown com frontmatter (schema do `dossier-template.md`) |
| INDEX atualizado | `../../kb/INDEX.md` | Tabela markdown |
| Sistema CONTEXT (se novo) | `../../kb/<tech>/<sistema>/CONTEXT.md` | Markdown |
