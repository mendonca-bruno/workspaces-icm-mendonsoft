# Polyglot Porter — Migração ICM Entre Linguagens

Este workspace recebe um **projeto-fonte ICM-compliant** em uma linguagem (Java/Spring Boot, .NET/ASP.NET ou Python) e produz um equivalente arquitetural em outra. Pipeline de 7 stages com 3 checkpoints humanos. Pares MVP: **Java↔.NET** e **Java↔Python**.

**Diferença vs. outros builders:** [workspace-builder-v2](../workspace-builder-v2/) cria workspaces ICM do zero, [workspace-retrofitter](../workspace-retrofitter/) adapta diretórios em workspaces ICM. O polyglot-porter **traduz a implementação** de um workspace ICM já existente para outra stack — ele *consome* ICM como pré-requisito.

**Acoplamento com retrofitter:** se o source não for ICM-compliant, stage 00 invoca o retrofitter automaticamente antes de prosseguir.

**Projetos de referência (opcional):** o usuário pode declarar `target_references[]` no questionnaire — paths para projetos na linguagem-alvo cujas convenções (layout, naming, DI, libs custom) devem guiar o target gerado. Têm precedência sobre golden-examples e archetypes default. Stage 02 destila as references em `reference-distillation.md` consumido por stages 04 e 05.

## Mapa de Pastas

```
polyglot-porter/
├── CLAUDE.md              ← você está aqui
├── CONTEXT.md             ← roteamento das 7 fases
├── README.md              ← como usar (input contract, fluxo)
├── _config/               ← L3 persistente (a "factory")
│   ├── language-archetypes.md
│   ├── pattern-equivalences/    (java↔dotnet, java↔python)
│   ├── idiom-mapping/           (null, errors, concurrency)
│   ├── anti-patterns-migration.md
│   ├── equivalence-rubric.md
│   ├── preflight-policy.md
│   └── reference-ingestion.md   (como destilar target_references do user)
├── stages/
│   ├── 00-preflight/      ← detecta ICM-compliance, auto-aciona retrofitter
│   ├── 01-source-analysis/ ← inventário do source
│   ├── 02-target-discovery/ ← stack-flavor alvo [CHECKPOINT #1]
│   ├── 03-pattern-mapping/ ← translation-map.csv [CHECKPOINT #2]
│   ├── 04-scaffold-target/ ← skeleton do target
│   ├── 05-translate/       ← tradução file-by-file
│   └── 06-validate-equivalence/ ← build + test + diff [CHECKPOINT #3]
├── shared/
│   ├── templates/         ← pom.xml, .csproj, pyproject.toml, CI YAMLs
│   └── schemas/           ← translation-map.schema.json, equiv-report.schema.json
├── references/
│   ├── golden-examples/   ← PetClinic, eShopOnWeb, full-stack-fastapi-template
│   └── conventions-reference.md
└── setup/
    └── questionnaire.md   ← perguntas mínimas antes de stage 00
```

## Triggers

| Keyword | Ação |
|---------|------|
| `setup` | Abrir questionnaire e coletar source/target/escopo |
| `port --source=<path> --target-lang=<java|dotnet|python>` | Iniciar pipeline em stage 00 |
| `status` | Mostrar progresso por stage (presença de outputs) |
| `approve --stage=<n>` | Liberar checkpoint correspondente |
| `rollback` | Apagar overlay do target (rollback total = remover pasta) |

## Roteamento

| Tarefa | Ir Para |
|--------|---------|
| Avaliar ICM-compliance do source e auto-acionar retrofitter | `stages/00-preflight/CONTEXT.md` |
| Inventariar source: módulos, frameworks, public surface | `stages/01-source-analysis/CONTEXT.md` |
| Escolher stack-flavor do target | `stages/02-target-discovery/CONTEXT.md` |
| Construir translation-map.csv | `stages/03-pattern-mapping/CONTEXT.md` |
| Gerar skeleton do target | `stages/04-scaffold-target/CONTEXT.md` |
| Traduzir código file-by-file | `stages/05-translate/CONTEXT.md` |
| Validar equivalência (build/test/diff) | `stages/06-validate-equivalence/CONTEXT.md` |

## O Que Carregar

| Tarefa | Carregar | NÃO Carregar |
|--------|----------|--------------|
| Stage 00 (preflight) | `_config/preflight-policy.md`, `setup/questionnaire.md` (respostas) | qualquer pattern-equivalences ou idiom-mapping |
| Stage 01 (analysis) | `_config/language-archetypes.md` (só seção do source-lang) | pattern-equivalences, templates |
| Stage 02 (discovery) | `_config/language-archetypes.md` (só seção do target-lang), `_config/reference-ingestion.md` (se `target_references[]` não-vazio), `target_references[]` paths, outputs de stage 01 | pattern-equivalences, templates |
| Stage 03 (mapping) | `_config/pattern-equivalences/<source>-to-<target>.md`, `_config/idiom-mapping/*`, `_config/anti-patterns-migration.md`, outputs 01-02 | language-archetypes, templates de scaffold |
| Stage 04 (scaffold) | `shared/templates/*` (só do target-lang), outputs de stage 03, `02/output/reference-distillation.md` (se existir) | pattern-equivalences, idiom-mapping |
| Stage 05 (translate) | `translation-map.csv` (versão pós-checkpoint #2), `_config/pattern-equivalences/<dir>.md`, golden-examples relevantes, `02/output/reference-distillation.md` (se existir) | language-archetypes, templates |
| Stage 06 (validate) | `_config/equivalence-rubric.md`, target build, source test fixtures | pattern-equivalences, idiom-mapping, golden-examples |

## Convenções

- **Caminho do source:** sempre absoluto, via `--source=<path>`. Nunca inferir.
- **Target overlay:** criado em `<source-name>-port-<target-lang>/`, sibling do source. Se já existir, sufixar com timestamp.
- **CSV é fonte de verdade:** stage 05 lê `translation-map.csv` da versão pós-checkpoint #2, nunca a versão inicial de stage 03.
- **Reversibilidade:** o overlay é totalmente isolado — rollback = `rm -rf <overlay>`. Nada é tocado no source.
- **Naming:** kebab-case em arquivos novos. Stages com zero-padded prefix (`00-`, `01-`).
- **Pares MVP:** Java↔.NET, Java↔Python. .NET↔Python e outros pares ficam para fase 2.
- **Precedência de convenções:** `target_flavor` explícito > `target_references` destiladas > golden-examples default > `language-archetypes.md`. Cada decisão em `target-decisions.md` cita a fonte aplicada.

## Hand-offs entre Stages

Sequencial 00 → 01 → 02 → 03 → 04 → 05 → 06. Cada stage lê apenas dos imediatamente anteriores. Edição manual de outputs entre stages é esperada (fluxo Unix: arquivo é a interface).

| Stage | Lê de | Escreve em |
|-------|-------|------------|
| 00 | `--source` + `setup/questionnaire.md` + `_config/preflight-policy.md` | `stages/00-preflight/output/` (+ possivelmente injeta scaffold ICM no source via retrofitter) |
| 01 | source ICM-compliant | `stages/01-source-analysis/output/` |
| 02 | stage 01 outputs + `_config/language-archetypes.md` | `stages/02-target-discovery/output/` |
| 03 | stage 01-02 outputs + `_config/pattern-equivalences/*` + `idiom-mapping/*` | `stages/03-pattern-mapping/output/` |
| 04 | stage 03 outputs (csv pós-checkpoint #2) + `shared/templates/*` | `<overlay>/` (skeleton) + `stages/04-scaffold-target/output/` |
| 05 | csv + scaffold de 04 + golden-examples | `<overlay>/` (código traduzido) + `stages/05-translate/output/` |
| 06 | overlay completo + source test corpus + `_config/equivalence-rubric.md` | `stages/06-validate-equivalence/output/` (relatórios) |

## Checkpoints Humanos

| Quando | Onde | O Que Aprovar |
|--------|------|---------------|
| Após stage 02 | `stages/02-target-discovery/output/target-decisions.md` | Stack-flavor alvo (versão do framework, ORM, DI container) reflete o desejo? |
| Após stage 03 | `stages/03-pattern-mapping/output/translation-map.csv` + `mapping-report.md` | Mapeamento por arquivo está correto? Criar `APPROVED.md` em `stages/03-pattern-mapping/output/` para liberar geração. |
| Após stage 06 | `stages/06-validate-equivalence/output/equivalence-report.md` | Aceitar nível atingido (L1/L2/L3/L4 da rubric)? |
