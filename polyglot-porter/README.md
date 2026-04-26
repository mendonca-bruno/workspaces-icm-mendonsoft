# Polyglot Porter

Workspace ICM para migração arquitetural entre linguagens. Recebe um projeto ICM-compliant em Java/Spring Boot, .NET/ASP.NET ou Python e produz o equivalente em outra linguagem MVP suportada.

## Pares Suportados (MVP)

- **Java ↔ .NET** — Spring Boot 3 ↔ ASP.NET 8 + EF Core
- **Java ↔ Python** — Spring Boot 3 ↔ FastAPI/Django + SQLAlchemy

Outros pares (.NET↔Python e variações) ficam para fase 2.

## Pré-requisito: Source ICM-compliant

Este workspace **assume** que o projeto-fonte está estruturado segundo a Interpreted Context Methodology (ICM): tem `CLAUDE.md`, `CONTEXT.md`, silos numerados ou similares. Se não estiver, o stage 00 invoca automaticamente o `workspace-retrofitter` no source para adicionar a estrutura ICM antes de prosseguir.

## Projetos de referência (opcional, recomendado)

Você pode passar `target_references[]` no questionnaire — uma lista de paths absolutos para projetos **na linguagem-alvo** cujas convenções (layout de pastas, naming, escolha de DI/ORM/validation, libs custom como MediatR/AutoMapper, padrões de testes) devem ser usadas como template.

Caso de uso típico: você já tem um projeto .NET interno na empresa que segue convenções estabelecidas, e quer que a migração de um Java legado siga essas mesmas convenções em vez do default genérico.

**Precedência (decisões em ordem):**
1. `target_flavor` explícito no questionnaire (sempre vence).
2. Destilado das `target_references` (consenso entre múltiplas references via maioria simples; divergências escalam para checkpoint #1).
3. Golden-example padrão da linguagem-alvo.
4. `_config/language-archetypes.md` (base teórica).

Stage 02 destila as references em `reference-distillation.md` (publicado em `stages/02-target-discovery/output/`); stages 04 e 05 consomem esse arquivo. Cada decisão em `target-decisions.md` cita explicitamente a fonte aplicada.

Detalhes em `_config/reference-ingestion.md`.

## Como Usar

1. **Setup.** Abrir `setup/questionnaire.md` e preencher a seção `## Suas respostas` com:
   - Caminho absoluto do source
   - Linguagem-fonte (`java` | `dotnet` | `python`)
   - Linguagem-alvo (`java` | `dotnet` | `python`)
   - Escopo (full | controllers-only | data-layer-only | etc.)
   - Restrições (pastas a ignorar, sensibilidade)

2. **Iniciar pipeline.** Dizer ao agente: `port --source=<path> --target-lang=<lang>`. O stage 00 começa.

3. **Acompanhar checkpoints.**
   - **Após stage 02:** revisar `stages/02-target-discovery/output/target-decisions.md` (versão de framework, ORM, DI). Editar se necessário.
   - **Após stage 03:** revisar `stages/03-pattern-mapping/output/translation-map.csv`. Editar livremente. Criar `APPROVED.md` em `stages/03-pattern-mapping/output/` para liberar geração de código.
   - **Após stage 06:** revisar `stages/06-validate-equivalence/output/equivalence-report.md` e aceitar/rejeitar o nível atingido.

4. **Status.** A qualquer momento, `status` mostra qual stage tem outputs preenchidos.

5. **Rollback.** O overlay do target é totalmente isolado — para descartar, basta `rm -rf <source>-port-<target-lang>/`.

## Garantias

- **Source nunca é modificado** (exceto por scaffold ICM via retrofitter no stage 00, que é gated por checkpoint do próprio retrofitter).
- **Target é overlay separado** em `<source-name>-port-<target-lang>/`, sibling do source.
- **Plain text como interface:** todos os artefatos intermediários são MD/CSV/JSON editáveis.
- **3 checkpoints humanos** em pontos de alto blast-radius (escolha de stack, mapeamento, sign-off final).

## Estrutura

Veja [CLAUDE.md](CLAUDE.md) para o mapa de pastas completo e [CONTEXT.md](CONTEXT.md) para roteamento de tarefas.

## Validação End-to-End

O workspace é validado contra o canary `spring-petclinic → ASP.NET`. Aceitação v1 exige nível **L2** da rubric: target compila e suite portada de testes passa contra fixtures compartilhadas.

## Arquitetura

Implementa os 5 princípios não-negociáveis do ICM: selectivity (não compressão), plain text universal, folder=pipe, human gate mandatório em outputs, factory-as-config. Inspirado em [`workspace-builder-v2`](../workspace-builder-v2/) (greenfield code) e [`workspace-retrofitter`](../workspace-retrofitter/) (brownfield ICM).
