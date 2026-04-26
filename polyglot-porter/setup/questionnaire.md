# Setup Questionnaire — Antes de Começar a Migração

Responda estas 8 perguntas. Suas respostas viram parte dos inputs de stage 00 (preflight) e stages 01–02. Salve este arquivo após preencher; o agente lê os campos abaixo da linha `## Suas respostas`.

---

## 1. Caminho do source

Caminho absoluto. Ex: `C:\Users\bruno\projects\petclinic` ou `/home/bruno/work/legacy-api`.

## 2. Linguagem-fonte

Uma de: `java` | `dotnet` | `python`.

Versão e framework principal (ex: `java 17 / spring-boot 3.2`, `dotnet 8 / aspnet-core`, `python 3.11 / django 5`).

## 3. Linguagem-alvo

Uma de: `java` | `dotnet` | `python`. Pares MVP suportados:

- Java ↔ .NET
- Java ↔ Python

Se preferir um stack-flavor específico (ex: `dotnet/minimal-api` vs `dotnet/mvc`, `python/fastapi` vs `python/django`), declare aqui — caso contrário, o stage 02 escolhe o default da archetype.

## 4. Escopo da migração

Escolha uma:

- `full` — migrar todo o source.
- `controllers-only` — migrar só camada de API/HTTP.
- `data-layer-only` — migrar só repositórios + entidades.
- `<custom>` — listar pacotes/módulos específicos.

## 5. Restrições — pastas a NUNCA tocar

Liste pastas/padrões que devem ser ignorados (além do default: `.git/`, `node_modules/`, `.venv/`, `target/`, `bin/`, `obj/`, `__pycache__/`, `dist/`, `build/`).

## 6. Sensibilidade — algo que o agente NÃO deve ler?

Arquivos com credenciais, dados pessoais, segredos. Liste padrões. O scan pode registrar nome e tamanho mas não lê conteúdo.

## 7. Test corpus — onde estão os testes do source?

Pastas que contêm testes a serem usados como oracle de equivalência no stage 06. Ex: `src/test/java`, `tests/`, `Tests/`.

## 8. Projetos de referência (opcional, mas recomendado)

Lista de projetos **na linguagem-alvo** que devem ser usados como template de convenções (layout, naming, padrões DI, escolha de libs, estilo de testes). Paths locais absolutos. Se a referência está num repo Git remoto, clone primeiro.

**Precedência:** convenções extraídas das suas referências sobrescrevem os defaults dos golden-examples e dos `_config/language-archetypes.md`. Se não passar referências, o workspace usa o golden-example padrão do `target_lang`.

Exemplo de uso típico:
- Source = projeto Java legado.
- Target = .NET.
- Reference = um projeto .NET interno da sua empresa que já segue convenções desejadas (organização de pastas, FluentValidation vs DataAnnotations, repositório próprio em vez de DbSet direto, etc.).

Pode passar **mais de uma**: o stage 02 destila um conjunto de convenções consensuais, marcando divergências para o checkpoint humano #1.

---

## Suas respostas

```yaml
source_path: ""
source_lang: ""              # java | dotnet | python
source_framework: ""         # ex: spring-boot 3.2
target_lang: ""              # java | dotnet | python
target_flavor: ""            # opcional, ex: dotnet/minimal-api, python/fastapi
scope: "full"                # full | controllers-only | data-layer-only | <custom>
ignore_paths: []
sensitive_patterns: []
test_paths: []
target_references: []        # lista de paths absolutos para projetos no target_lang a usar como template
                             # ex: ["C:/dev/internal-api", "C:/dev/reference-service"]
```

---

*Após preencher, diga `port --source=<path> --target-lang=<lang>` para iniciar stage 00.*
