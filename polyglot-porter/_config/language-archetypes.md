# Language Archetypes — Estrutura Idiomática por Linguagem

Catálogo do que é "idiomático" em cada linguagem MVP. Stage 01 usa a seção da source-lang para inventariar; stage 02 usa a seção da target-lang para travar stack-flavor. Seções não relevantes ao par atual NÃO devem ser carregadas.

---

## Java / Spring Boot

### Stack-flavors disponíveis

| Flavor | Descrição | Default |
|---|---|---|
| `java/spring-boot-3` | Spring Boot 3.x + Spring Web MVC + Spring Data JPA + Hibernate | ✅ |
| `java/spring-boot-2` | Spring Boot 2.x (legado, javax.* em vez de jakarta.*) | — |
| `java/quarkus` | Quarkus + RESTEasy + Panache (não MVP) | — |

### Layout idiomático

```
project/
├── pom.xml | build.gradle.kts
├── src/main/java/<base.package>/
│   ├── Application.java          (@SpringBootApplication entrypoint)
│   ├── controller/                (@RestController)
│   ├── service/                   (@Service, business logic)
│   ├── repository/                (@Repository, extends JpaRepository)
│   ├── domain/ ou model/          (@Entity, JPA models)
│   ├── dto/                       (request/response objects)
│   ├── config/                    (@Configuration beans)
│   └── exception/                 (custom exceptions + @ControllerAdvice)
├── src/main/resources/
│   ├── application.yml            (config)
│   └── db/migration/              (Flyway, opcional)
└── src/test/java/<base.package>/
    └── <espelho de main>/         (@SpringBootTest, @WebMvcTest, @DataJpaTest)
```

### Componentes-chave

| Categoria | Anotação/Pattern | Uso |
|---|---|---|
| API endpoint | `@RestController` + `@GetMapping`/`@PostMapping`/etc. | Camada HTTP |
| DI container | `@Autowired` (preferir injeção via construtor) | Resolução de dependências |
| ORM | `@Entity`, `@Id`, `@OneToMany`, `JpaRepository` | Persistência |
| Validation | `@Valid` + `jakarta.validation.constraints.*` | Bean Validation |
| Error handling | `@ControllerAdvice` + `@ExceptionHandler` | Exception mapping global |
| Async | `CompletableFuture<T>`, `@Async` | Concorrência |
| Config | `@ConfigurationProperties`, `application.yml` | Externalized config |

### Testes

| Tipo | Anotação | Comando |
|---|---|---|
| Unit | JUnit 5 + Mockito | `mvn test` |
| Web slice | `@WebMvcTest` + `MockMvc` | idem |
| Repository slice | `@DataJpaTest` + `Testcontainers` | idem |
| Integration | `@SpringBootTest` | idem |

---

## .NET / ASP.NET

### Stack-flavors disponíveis

| Flavor | Descrição | Default |
|---|---|---|
| `dotnet/minimal-api` | .NET 8 + Minimal API + EF Core | ✅ |
| `dotnet/mvc` | .NET 8 + MVC controllers + EF Core | — |
| `dotnet/web-api` | .NET 8 + ApiController atributos + EF Core | alternativa |

### Layout idiomático

```
ProjectName/
├── ProjectName.sln
├── src/ProjectName/
│   ├── ProjectName.csproj
│   ├── Program.cs                  (entrypoint, builder.Services.Add*())
│   ├── Controllers/                ([ApiController])
│   ├── Services/                   (business logic, registrado em DI)
│   ├── Data/
│   │   ├── AppDbContext.cs         (DbContext + DbSet<>)
│   │   └── Migrations/             (EF Core migrations)
│   ├── Models/ ou Domain/          (entity classes)
│   ├── Dtos/                       (request/response records)
│   ├── Configuration/              (IOptions<T>)
│   └── Exceptions/                 (custom + middleware)
├── src/ProjectName.Contracts/      (opcional, DTOs públicos)
└── tests/ProjectName.Tests/
    └── <espelho>/                  (xUnit + Moq + WebApplicationFactory<>)
```

### Componentes-chave

| Categoria | Atributo/Pattern | Uso |
|---|---|---|
| API endpoint | `[ApiController]` + `[HttpGet]`/`[HttpPost]` ou `app.MapGet()` | Camada HTTP |
| DI container | Constructor injection (registrado via `builder.Services.AddScoped<>`) | Resolução |
| ORM | `DbContext`, `DbSet<T>`, atributos EF (`[Key]`, `[ForeignKey]`) | Persistência |
| Validation | `DataAnnotations` ou `FluentValidation` | Bean Validation |
| Error handling | `IExceptionHandler` (NET 8) ou middleware customizado | Exception mapping |
| Async | `Task<T>`, `async`/`await` (default em I/O) | Concorrência |
| Config | `IOptions<T>` + `appsettings.json` | Externalized config |

### Testes

| Tipo | Framework | Comando |
|---|---|---|
| Unit | xUnit + Moq ou NSubstitute | `dotnet test` |
| Web/integration | `WebApplicationFactory<Program>` | idem |
| EF Core | InMemory provider ou SQLite in-memory ou Testcontainers | idem |

---

## Python

### Stack-flavors disponíveis

| Flavor | Descrição | Default |
|---|---|---|
| `python/fastapi` | FastAPI + Pydantic + SQLAlchemy 2 + Alembic | ✅ |
| `python/django` | Django 5 + DRF + Django ORM | alternativa |
| `python/flask` | Flask + SQLAlchemy (mais antigo, não MVP) | — |

### Layout idiomático (FastAPI)

```
project/
├── pyproject.toml
├── src/<package>/
│   ├── __init__.py
│   ├── main.py                     (FastAPI app + lifespan)
│   ├── api/
│   │   └── v1/
│   │       ├── routes/             (APIRouter por recurso)
│   │       └── deps.py             (Depends() para DI)
│   ├── services/                   (business logic)
│   ├── db/
│   │   ├── base.py                 (DeclarativeBase)
│   │   ├── session.py              (engine + sessionmaker)
│   │   └── models/                 (entity classes)
│   ├── repositories/               (data access)
│   ├── schemas/                    (Pydantic models)
│   ├── core/
│   │   ├── config.py               (Settings via pydantic-settings)
│   │   └── exceptions.py
│   └── alembic/                    (migrations)
└── tests/
    └── <espelho>/                  (pytest + httpx + pytest-asyncio)
```

### Layout idiomático (Django)

```
project/
├── manage.py
├── pyproject.toml
├── config/                          (project-level)
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py | asgi.py
└── apps/
    └── <app>/
        ├── models.py
        ├── views.py | viewsets.py  (DRF ViewSet)
        ├── serializers.py
        ├── urls.py
        ├── admin.py
        └── tests/
```

### Componentes-chave

| Categoria | FastAPI pattern | Django pattern |
|---|---|---|
| API endpoint | `@router.get()`, `@router.post()` | `ViewSet` + `urls.py` |
| DI container | `Depends()` | nativo via settings + apps |
| ORM | SQLAlchemy 2 (`Mapped[]`, `mapped_column`) | Django ORM (`models.Model`) |
| Validation | Pydantic (`BaseModel`) | DRF Serializers |
| Error handling | `HTTPException` + exception handlers | `APIException` + custom handler |
| Async | `async def` + `asyncio` | Django async views (4.x+) |
| Config | `pydantic-settings BaseSettings` | `settings.py` + env vars |

### Testes

| Framework | Comando |
|---|---|
| pytest + httpx (FastAPI) | `pytest` |
| pytest-django (Django) | `pytest` |
| factory-boy para fixtures | — |
