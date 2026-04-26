# Stage 02: Target Discovery

Travar stack-flavor do target: versão de framework, ORM, DI container, layout idiomático. Decisão de alto blast-radius — termina em **checkpoint humano #1**. Se o usuário forneceu `target_references`, destilar convenções delas (precedência sobre archetypes/golden defaults).

## Inputs

| Fonte | Arquivo / Localização | Escopo | Por quê |
|-------|----------------------|--------|---------|
| Setup | `../../setup/questionnaire.md` | `target_lang`, `target_flavor`, `target_references[]` | Restrições + references |
| Stage 01 | `../01-source-analysis/output/source-inventory.md` | Completo | Frameworks usados, escala |
| Stage 01 | `../01-source-analysis/output/dependencies.json` | Completo | Dependências do source que precisam de equivalente |
| Config | `../../_config/language-archetypes.md` | Apenas seção da `target_lang` | Catálogo de stack-flavors |
| Config | `../../_config/equivalence-rubric.md` | Seção L1 (build commands) | Comando de build do target |
| Config | `../../_config/reference-ingestion.md` | Completo (apenas se `target_references` não-vazio) | Como destilar convenções |
| User refs | Cada path em `target_references[]` | Walk até 3 níveis, filtrando dirs default | Fonte de convenções com precedência |
| References | `../../references/golden-examples/` (apenas o arquivo do target) | Completo | Fallback se sem user refs |

## Process

1. Se `target_flavor` foi explicitamente declarado no questionnaire, registrar com precedência máxima.
2. **Se `target_references[]` não-vazio:**
   1. Para cada path: validar que existe, é diretório, e contém build file da `target_lang`. Skipar inválidos com warning em `reference-distillation.md` § "Skipped".
   2. Para cada reference válida, extrair as 7 dimensões definidas em `reference-ingestion.md`. Escrever `output/references/<ref-name>-profile.md`.
   3. Destilar consensual entre references usando estratégia de maioria simples (ver `reference-ingestion.md` § "Como destilar"). Marcar divergências para escalation.
   4. Escrever `reference-distillation.md` agregando: padrões consensuais, divergências, references skipadas, libs/pacotes não-default detectados, naming, layout.
3. **Se `target_references[]` vazio:** carregar golden-example default da `target_lang` e usá-lo como base.
4. Mapear cada framework do source para framework target, **respeitando precedência** definida em `reference-ingestion.md` § "Precedência":
   - Se reference-distillation define algo, usar.
   - Senão, golden-example.
   - Senão, archetype default.
5. Mapear cada dependência relevante do source para equivalente target. Se references usam libs específicas (ex: MediatR, FluentValidation, AutoMapper), preferir essas equivalências.
6. Determinar layout de pastas: do destilado se existir, senão do archetype.
7. Travar versões: framework principal, runtime/SDK, build tool. Se references têm versões mais recentes/antigas que archetype default, registrar trade-off.
8. Identificar gaps: dependências do source sem equivalente claro, OU divergências entre references não resolvíveis por maioria.
9. Escrever `target-decisions.md` com tabela de decisões. **Cada linha cita a fonte da decisão** (`target_flavor` | `reference[i]` | `golden-example` | `archetype-default`).
10. Listar trade-offs significativos e escalations no topo do arquivo.

## Checkpoints

| Quando | Onde | Aprovação |
|---|---|---|
| Após escrita de `target-decisions.md` + `reference-distillation.md` (se aplicável) | Próprios arquivos | Usuário revisa e edita versões/flavors. Sem APPROVED.md, mas pipeline pausa até confirmação verbal. Atenção especial: divergências entre references devem ser resolvidas explicitamente. |

## Audit

| Verificação | Aprovação |
|-------------|-----------|
| Cada framework do source mapeado | Sem framework órfão |
| Versões travadas explicitamente | Sem "latest" ambíguo |
| Layout de pastas referenciado | Path explícito do source de decisão |
| Cada decisão tem `source` declarado | Coluna "fonte" preenchida em `target-decisions.md` |
| References inválidas registradas | Seção "Skipped" em `reference-distillation.md` |
| Divergências escaladas | Seção "Open Questions" para checkpoint |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Decisões de stack | `output/target-decisions.md` | MD: tabela framework→equivalente + versão + **fonte** + justificativa + gaps |
| Mapa de dependências | `output/dependency-map.json` | JSON: {source_dep, target_dep, version, confidence, source: "reference\|golden\|archetype"} |
| Destilado de references | `output/reference-distillation.md` | MD: consensos + divergências + skipped (apenas se `target_references` não-vazio) |
| Profiles individuais | `output/references/<ref-name>-profile.md` | MD: 7 dimensões por reference |

## Hand-off

Stage 03 lê os outputs + `_config/pattern-equivalences/<source>-to-<target>.md` + `idiom-mapping/*` para construir translation-map.csv. Stages 04 e 05 também consultam `reference-distillation.md` para layout/naming/lib choice.
