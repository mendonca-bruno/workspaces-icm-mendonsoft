# Templates — Polyglot Porter

Esqueletos paramétricos por stack-flavor target. Stage 04 (scaffold-target) lê apenas a pasta correspondente ao `target_lang` decidido em stage 02.

## Estrutura

```
templates/
├── java/                  (Spring Boot 3 + JPA)
│   ├── pom.xml.tmpl
│   ├── Application.java.tmpl
│   └── application.yml.tmpl
├── dotnet/                (ASP.NET 8 + EF Core)
│   ├── Project.csproj.tmpl
│   ├── Program.cs.tmpl
│   └── appsettings.json.tmpl
├── python/                (FastAPI + SQLAlchemy 2)
│   ├── pyproject.toml.tmpl
│   ├── main.py.tmpl
│   └── config.py.tmpl
└── ci/                    (GitHub Actions YAMLs)
    ├── github-java.yml.tmpl
    ├── github-dotnet.yml.tmpl
    └── github-python.yml.tmpl
```

## Placeholders

Sintaxe `{{VAR}}` para substituição direta, `{{?FLAG}}...{{/FLAG}}` para blocos condicionais. Mesma sintaxe do `_core/placeholder-syntax.md` do ICM.

Variáveis comuns por stack:

| Stack | Vars típicas |
|---|---|
| Java | `SPRING_BOOT_VERSION`, `JAVA_VERSION`, `GROUP_ID`, `ARTIFACT_ID`, `BASE_PACKAGE`, `DB_DRIVER_GROUP`, `DB_DRIVER_ARTIFACT`, `USE_FLYWAY`, `USE_TESTCONTAINERS` |
| .NET | `DOTNET_TARGET`, `ROOT_NAMESPACE`, `ASSEMBLY_NAME`, `EFCORE_VERSION`, `DB_PROVIDER_PACKAGE`, `DB_PROVIDER_METHOD`, `DBCONTEXT_NAME`, `USE_FLUENT_VALIDATION` |
| Python | `PROJECT_NAME`, `VERSION`, `PYTHON_VERSION`, `FASTAPI_VERSION`, `PACKAGE_NAME`, `DB_DRIVER_PACKAGE`, `RUFF_TARGET`, `ENV_PREFIX` |

Stage 04 deriva os valores de `target-decisions.md` e `dependency-map.json` (ambos de stage 02).

## Convenção

Templates são **mínimos**. Não trazem código de domínio (entities, controllers concretos) — esse trabalho é do stage 05. Apenas:
- Build file
- Entrypoint (Application.java / Program.cs / main.py)
- Config (application.yml / appsettings.json / config.py)
- CI workflow

Pastas de domínio (`controller/`, `service/`, `entity/` etc.) são criadas vazias com `.gitkeep`.
