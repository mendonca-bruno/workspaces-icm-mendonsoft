# Golden Example — eShopOnWeb

Repositório de referência canônica para .NET/ASP.NET. Stage 02 cita como template; stage 06 usa shape para cross-check.

## URL

https://github.com/dotnet-architecture/eShopOnWeb

## Stack

- .NET 8
- ASP.NET Core
- EF Core
- SQL Server (default) / SQLite (dev)
- xUnit + Moq

## Expected shape

```
eShopOnWeb/
├── eShopOnWeb.sln
├── src/
│   ├── ApplicationCore/                 (domain, sem dependências de infra)
│   │   ├── Entities/
│   │   ├── Interfaces/
│   │   ├── Services/
│   │   └── Specifications/
│   ├── Infrastructure/                  (EF Core, repositories concretos)
│   │   ├── Data/
│   │   │   ├── CatalogContext.cs
│   │   │   ├── Config/
│   │   │   └── Migrations/
│   │   ├── Identity/
│   │   └── Services/
│   ├── Web/                             (ASP.NET MVC + Razor Pages)
│   │   ├── Controllers/
│   │   ├── Pages/
│   │   └── Program.cs
│   └── PublicApi/                       (ASP.NET Core Web API)
│       ├── Controllers/
│       └── Program.cs
└── tests/
    ├── UnitTests/
    ├── IntegrationTests/
    └── FunctionalTests/                 (WebApplicationFactory<Program>)
```

## Componentes representativos

| Conceito | Arquivo | Padrão |
|---|---|---|
| Entity | `ApplicationCore/Entities/CatalogItem.cs` | classe simples + EF config separada |
| EF config | `Infrastructure/Data/Config/CatalogItemConfiguration.cs` | `IEntityTypeConfiguration<T>` (fluent API) |
| DbContext | `Infrastructure/Data/CatalogContext.cs` | `DbSet<CatalogItem>` + `OnModelCreating` |
| Repository pattern | `Infrastructure/Data/EfRepository.cs` | `IRepository<T>` + `EfRepository<T>` |
| Service | `ApplicationCore/Services/BasketService.cs` | classe + interface, registrado via DI |
| Web API controller | `PublicApi/Controllers/CatalogController.cs` | `[ApiController]` + `[Route]` |
| Integration test | `tests/FunctionalTests/Web/...` | `WebApplicationFactory<Program>` |

## Casos canários para L3 equivalence

| Caso | Endpoint | Espera |
|---|---|---|
| Listar catalog items | `GET /api/catalog-items` | 200 + lista paginada |
| Buscar item por ID | `GET /api/catalog-items/{id}` | 200 + item |
| Criar item | `POST /api/catalog-items` | 201 + Location header |
| Adicionar ao basket | `POST /api/basket-items` | 200 + basket atualizado |

## Por que este exemplo

- Arquitetura em camadas bem separadas (Application/Domain core vs Infrastructure vs Web).
- Exemplo oficial do .NET architecture team.
- Tem 2 frontends (MVC e Web API) — útil para mapear de Spring MVC e Spring REST.
- Padrão de testes funcionais com `WebApplicationFactory`.

## Caveat para migração

- eShopOnWeb usa Razor Pages no projeto Web. Se source for Spring REST puro, ignorar Razor e usar apenas `PublicApi/` como template.
- Specifications pattern (`ApplicationCore/Specifications/`) é mais avançado que JpaRepository padrão. Pode ser overkill para migrar PetClinic. Default flavor pode usar repository simples.

## Não vendor

URL + esta descrição apenas.
