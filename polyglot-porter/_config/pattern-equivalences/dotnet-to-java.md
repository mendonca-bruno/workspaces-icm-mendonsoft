# Pattern Equivalences — .NET/ASP.NET → Java/Spring Boot

Mapeamento canônico de padrões. Inverso de [`java-to-dotnet.md`](java-to-dotnet.md). Stage 03 lê esta tabela ao construir o `translation-map.csv`.

## Web layer

| .NET | Java/Spring equivalente | Notas |
|---|---|---|
| `[ApiController]` + `[Route("api/[controller]")]` | `@RestController` + `@RequestMapping("/api/foo")` | Spring não infere por nome |
| `[HttpGet("{id}")]` | `@GetMapping("/{id}")` | — |
| `[HttpPost]` + `[FromBody]` | `@PostMapping` + `@RequestBody` | — |
| `[FromRoute]` | `@PathVariable` | — |
| `[FromQuery]` | `@RequestParam` | — |
| `[FromHeader]` | `@RequestHeader` | — |
| `IActionResult` / `Results.Ok(...)` | `ResponseEntity<T>` | — |
| `IExceptionHandler` (.NET 8) | `@ControllerAdvice` + `@ExceptionHandler` | — |
| Minimal API `app.MapGet(...)` | Sem equivalente direto; usar `@RestController` | Java não tem Minimal-style |

## DI / lifecycle

| .NET | Java/Spring equivalente | Notas |
|---|---|---|
| `services.AddScoped<IFoo, Foo>()` | `@Service` na classe `Foo` (que implementa `IFoo`) | Spring usa `@ComponentScan` |
| `services.AddSingleton<>()` | `@Service` (default scope é singleton no Spring) | — |
| `services.AddTransient<>()` | `@Scope("prototype")` | Raramente usado |
| Constructor injection | `@Autowired` (ctor preferido em projetos modernos) | — |
| Keyed services (`AddKeyedScoped`) | `@Qualifier("name")` | — |
| `IOptions<T>` + `appsettings.json` | `@ConfigurationProperties("app")` + `application.yml` | — |

## Persistência (EF Core → JPA)

| EF Core | JPA | Notas |
|---|---|---|
| classe + `OnModelCreating` ou `[Table]` | `@Entity` + `@Table(name = "x")` | — |
| `[Key]` (convenção `Id`) | `@Id` | — |
| `[DatabaseGenerated(Identity)]` | `@GeneratedValue(strategy = IDENTITY)` | — |
| `[Column("x")]` | `@Column(name = "x")` | — |
| `.HasOne(...).WithMany(...).OnDelete(Cascade)` | `@OneToMany(cascade = ALL, mappedBy = "...")` | Cuidado com cascade semantics |
| `.HasForeignKey("XId")` | `@JoinColumn(name = "x_id")` | — |
| `DbSet<T>` em `AppDbContext` | `JpaRepository<T, Long>` | Spring Data dá CRUD pronto |
| LINQ query | JPQL ou `Specification<T>` | — |
| `using var tx = await db.Database.BeginTransactionAsync()` | `@Transactional` (no método ou classe) | — |
| EF Migrations | Flyway ou Liquibase | Modelo diferente |

## Async / Concurrency

| .NET | Java | Notas |
|---|---|---|
| `Task<T>` | `CompletableFuture<T>` | Direto |
| `async`/`await` (em I/O) | `@Async` (Spring) ou `CompletableFuture` manual | Java assíncrono é mais verboso |
| `IAsyncEnumerable<T>` | `Flux<T>` (Project Reactor / WebFlux) ou `Stream<T>` | Reativo se necessário |
| `Channel<T>` | `BlockingQueue<T>` ou `Sinks` (Reactor) | — |
| `CancellationToken` | Não há equivalente puro; usar `Future.cancel()` ou `Thread.interrupt()` | Cuidado |

## Validation

| .NET | Java | Notas |
|---|---|---|
| `[Required]` | `@NotNull` | — |
| `[StringLength(max, MinimumLength = min)]` | `@Size(min, max)` | — |
| `[EmailAddress]` | `@Email` | — |
| `[RegularExpression(regex)]` | `@Pattern(regex)` | — |
| `[Range(min, max)]` | `@Min(min)` + `@Max(max)` | — |
| FluentValidation | `ConstraintValidator` custom | — |

## Logging

| .NET | Java | Notas |
|---|---|---|
| `ILogger<Foo>` injected | `LoggerFactory.getLogger(Foo.class)` (SLF4J) | — |
| `_logger.LogInformation("msg {Arg}", arg)` | `log.info("msg {}", arg)` | — |

## Testing

| .NET | Java | Notas |
|---|---|---|
| `[Fact]` (xUnit) | `@Test` (JUnit 5) | — |
| `[Theory]` + `[InlineData]` | `@ParameterizedTest` + `@ValueSource` | — |
| Moq `Mock<IFoo>` | Mockito `mock(Foo.class)` | — |
| `WebApplicationFactory<Program>` | `@SpringBootTest` | Integration |
| EF InMemory / SQLite-in-memory | `@DataJpaTest` + Testcontainers | — |

## Build / Project file

| .NET | Java | Notas |
|---|---|---|
| `.csproj` + `.sln` | `pom.xml` (Maven) ou `build.gradle.kts` | — |
| `<PackageReference>` | `<dependency>` Maven | — |
| `dotnet build` | `mvn package` | — |
| `dotnet test` | `mvn test` | — |

## Application config

| .NET | Java | Notas |
|---|---|---|
| `appsettings.json` | `application.yml` ou `application.properties` | YAML mais comum em Spring |
| `services.Configure<AppOptions>(...)` | `@ConfigurationProperties("app")` | — |
| `ASPNETCORE_ENVIRONMENT=Development` | `@Profile("dev")` + `application-dev.yml` | — |
