# Reference Ingestion — Como Destilar Convenções de Projetos-Referência

O stage 02 (target-discovery) lê os `target_references` do questionnaire e extrai convenções concretas a serem aplicadas no target gerado. Este documento define **o que** extrair, **como** extrair, e **como resolver conflitos** quando há múltiplas references.

## O que extrair (per-reference)

Para cada projeto-referência, o agente produz um perfil estruturado com 7 dimensões:

| # | Dimensão | Como detectar |
|---|---|---|
| 1 | **Layout de pastas** | Tree até 3 níveis. Identificar pastas-chave (controllers/, services/, etc.) e nomes não-idiomáticos (ex: `application/`, `domain/`, `presentation/`) |
| 2 | **Naming convention** | Sample de 10–20 arquivos: classes, namespaces/packages, casing de pastas |
| 3 | **DI / lifecycle** | Como serviços são registrados (extensão method? `Program.cs` direto? `@Configuration` separada? `Depends()` factories agrupadas?) |
| 4 | **Persistência** | ORM usado, padrão de Repository (DbSet direto, IRepository<T>, Specification, EF Generic Repository) |
| 5 | **Validation** | DataAnnotations vs FluentValidation; Pydantic vs `@field_validator`; Bean Validation vs Hibernate Validator manual |
| 6 | **Error handling** | Middleware? `@ControllerAdvice`? `IExceptionHandler` (.NET 8)? exception_handler decorator? Estrutura de erro de resposta (RFC 7807 ProblemDetails? custom?) |
| 7 | **Testing** | Framework (xUnit/NUnit/MSTest, JUnit/Spock, pytest/unittest), padrão de fixtures, factory pattern, uso de Testcontainers |

Adicionalmente, listar:
- **Libs/pacotes não-default** — qualquer pacote em build file que não esteja no archetype default (ex: `MediatR`, `AutoMapper`, `MapStruct`, `factory-boy`).
- **Custom analyzers / linters** — `EditorConfig`, `.editorconfig`, `ruff.toml`, `pyproject.toml [tool.ruff]`, `pmd-rules.xml`.
- **Padrões CI** — workflows YAML detectados.

Output por reference: `<reference-name>-profile.md` em `stages/02-target-discovery/output/references/`.

## Como extrair

Sequência aplicada pelo agente:

1. Validar que cada path em `target_references` existe e é diretório. Skipar inválidos com warning.
2. Verificar que cada reference é da `target_lang` (procurar `pom.xml` se java, `*.csproj`/`*.sln` se dotnet, `pyproject.toml` se python). Skipar com warning se mismatch.
3. Walk até 3 níveis. Filtrar `.git/`, `node_modules/`, `target/`, `bin/`, `obj/`, `.venv/`, `__pycache__/`, `dist/`, `build/`.
4. Para dimensão 1: gerar tree.
5. Para dimensão 2: ler 10–20 arquivos representativos da maior camada (ex: controllers/) e tabular naming.
6. Para dimensões 3–7: heurística por anotação/atributo/import + busca de strings-chave (`AddScoped`, `@Configuration`, `Depends(`, `@ControllerAdvice`, etc.).
7. Escrever profile.

Não é necessário compilar nem rodar testes da reference — análise estática é suficiente.

## Como destilar (consolidar múltiplas references)

Quando há ≥ 2 references, o stage 02 produz um **destilado consensual** em `reference-distillation.md`:

| Estratégia | Aplicação |
|---|---|
| **Maioria simples** | Para cada dimensão, pegar o padrão mais frequente entre as references. Ex: 2 de 3 references usam FluentValidation → adotar FluentValidation. |
| **Marcação de divergência** | Onde references discordam (ex: 1 usa DI manual, 1 usa Autofac), listar opções e escalar para checkpoint #1. |
| **Override do questionnaire** | Se `target_flavor` foi declarado explicitamente, ele tem precedência sobre o destilado. |

Se há apenas 1 reference, o destilado = profile dela (sem voting necessário).

## Precedência (resolução final de convenções)

Ordem de prioridade, do mais alto ao mais baixo:

1. **`target_flavor` explícito no questionnaire** — sempre vence.
2. **`target_references` (destilado)** — se passadas pelo usuário.
3. **`references/golden-examples/<arquivo-do-target>.md`** — fallback default.
4. **`_config/language-archetypes.md`** — base teórica.

Toda decisão em `target-decisions.md` cita explicitamente o nível de precedência aplicado (ex: "DI: FluentValidation — fonte: target_references[0] internal-api/").

## Output esperado

```
stages/02-target-discovery/output/
├── target-decisions.md          (decisões finais com source de cada uma)
├── dependency-map.json
├── reference-distillation.md    (NOVO — consolidação das references)
└── references/                  (NOVO — perfil por reference)
    ├── <ref-name-1>-profile.md
    └── <ref-name-2>-profile.md
```

## Anti-patterns

1. **Cherry-picking inconsistente.** Não escolher "naming da ref-A + DI da ref-B + persistência da ref-C". Aplicar maioria simples ou escalar.
2. **Ignorar reference inválida silenciosamente.** Sempre logar em `reference-distillation.md` quando uma reference foi rejeitada (lang mismatch, path inválido).
3. **Vendor a reference.** Não copiar arquivos da reference para dentro do target overlay. A reference é só fonte de convenção.
4. **Confiar em reference muito pequena.** Se a reference tem < 5 arquivos no target_lang, registrar warning "low-signal reference" e cair para precedência inferior.
5. **Sobrescrever decisão arquitetural sem checkpoint.** Mudanças de stack-flavor (ex: minimal-api → MVC) inferidas da reference SEMPRE escalam para checkpoint #1.

## Edge cases

- **Reference path não existe.** Skipar; registrar em `reference-distillation.md` § "Skipped".
- **Reference está em lang diferente.** Skipar com warning ("expected `target_lang=dotnet`, found Java project at <path>").
- **Múltiplas references com layouts radicalmente diferentes.** Escalar para checkpoint #1 com proposta. Não fundir layouts artificialmente.
- **Reference é um monorepo.** Identificar subprojetos do target_lang e tratar cada um como reference separada.
