# Pattern Equivalences — Java/Spring Boot → .NET/ASP.NET

Mapeamento canônico de padrões. Stage 03 lê esta tabela ao construir o `translation-map.csv`. Stage 05 a usa como referência durante tradução por arquivo.

## Web layer

| Java/Spring | .NET equivalente | Notas |
|---|---|---|
| `@RestController` + `@RequestMapping("/api")` | `[ApiController]` + `[Route("api/[controller]")]` ou `app.MapGroup("/api")` (Minimal) | Default flavor: Minimal API |
| `@GetMapping("/{id}")` | `[HttpGet("{id}")]` ou `app.MapGet("/{id}", ...)` | — |
| `@PostMapping` + `@RequestBody` | `[HttpPost]` + `[FromBody]` | `[FromBody]` é default em ApiController |
| `@PathVariable` | `[FromRoute]` | — |
| `@RequestParam` | `[FromQuery]` | — |
| `@RequestHeader` | `[FromHeader]` | — |
| `ResponseEntity<T>` | `IActionResult` ou `Results.Ok(t)` (Minimal) | — |
| `@ControllerAdvice` + `@ExceptionHandler` | `IExceptionHandler` (.NET 8) ou middleware customizado | — |
| `@Valid` + bean validation | `[ApiController]` faz auto-400 com DataAnnotations | — |

## DI / lifecycle

| Java/Spring | .NET equivalente | Notas |
|---|---|---|
| `@Service` | `services.AddScoped<IFoo, Foo>()` em Program.cs | Sem atributo no tipo |
| `@Component` | `services.AddTransient<>()` ou `AddScoped<>()` | Mesma coisa |
| `@Repository` | idem | Convenção, não atributo |
| `@Configuration` + `@Bean` | `services.AddSingleton<>(provider => ...)` | Lambda factory |
| `@Autowired` (ctor) | Constructor injection (sem atributo) | C# 12 primary ctor preferido |
| `@Qualifier("name")` | `services.AddKeyedScoped<>("name", Type)` (NET 8+) | Keyed services |
| `@Value("${app.foo}")` | `IOptions<MyOptions>` + `appsettings.json` | Strongly-typed |

## Persistência (JPA → EF Core)

| JPA | EF Core | Notas |
|---|---|---|
| `@Entity` | classe normal registrada em `DbContext.OnModelCreating` ou `[Table]` atributo | EF tem fluent + atributos |
| `@Id` + `@GeneratedValue(IDENTITY)` | `[Key]` (convenção ID) ou `.HasKey()`; `.UseIdentityColumn()` | — |
| `@Column(name = "x")` | `[Column("x")]` ou `.Property(p => p.X).HasColumnName("x")` | — |
| `@OneToMany(mappedBy = "x", cascade = ALL)` | `.HasOne(x).WithMany(y => y.Items).OnDelete(DeleteBehavior.Cascade)` | — |
| `@ManyToOne` | `.HasOne(x).WithMany()` | — |
| `@JoinColumn(name = "x_id")` | `.HasForeignKey("XId")` ou `[ForeignKey]` | — |
| `JpaRepository<T, Long>` | `DbSet<T>` em `AppDbContext` + Repository pattern manual | EF não tem JpaRepository pronto |
| `@Query("JPQL")` | `.FromSql()` ou LINQ | LINQ preferido |
| `@Transactional` | `using var tx = await db.Database.BeginTransactionAsync()` | Sem atributo class-level limpo |
| Hibernate `@CreationTimestamp` | EF `[DatabaseGenerated(DatabaseGeneratedOption.Computed)]` ou interceptor | — |
| Flyway migrations | EF Core Migrations (`dotnet ef migrations add X`) | Modelo diferente |

## Async / Concurrency

| Java | .NET | Notas |
|---|---|---|
| `CompletableFuture<T>` | `Task<T>` | Direto |
| `@Async` (Spring) | `async`/`await` em métodos `Task<>` | .NET é async-by-default em I/O |
| `Mono<T>` / `Flux<T>` (WebFlux) | `IAsyncEnumerable<T>` ou `Task<T>` | Reativo só se necessário |
| `ExecutorService` | `Task.Run()` ou `Channel<T>` | — |

## Validation

| Java | .NET | Notas |
|---|---|---|
| `@NotNull` (Bean Validation) | `[Required]` (DataAnnotations) | — |
| `@Size(min, max)` | `[StringLength(max, MinimumLength = min)]` | — |
| `@Email` | `[EmailAddress]` | — |
| `@Pattern(regex)` | `[RegularExpression(regex)]` | — |
| `@Min/@Max` | `[Range(min, max)]` | — |
| Custom `ConstraintValidator` | Custom `ValidationAttribute` ou FluentValidation | FluentValidation preferido |

## Logging

| Java | .NET | Notas |
|---|---|---|
| SLF4J `LoggerFactory.getLogger(Foo.class)` | `ILogger<Foo>` injected | Idiomático |
| `log.info("msg {}", arg)` | `_logger.LogInformation("msg {Arg}", arg)` | Structured |

## Testing

| Java | .NET | Notas |
|---|---|---|
| JUnit 5 `@Test` | `[Fact]` (xUnit) | — |
| `@ParameterizedTest` + `@ValueSource` | `[Theory]` + `[InlineData]` | — |
| Mockito `mock(Foo.class)` | `Mock<IFoo>` (Moq) ou `Substitute.For<IFoo>()` (NSubstitute) | NSubstitute mais limpo |
| `@SpringBootTest` | `WebApplicationFactory<Program>` | Integration |
| `@WebMvcTest` | `WebApplicationFactory<>` com config customizada | — |
| `@DataJpaTest` | EF InMemory ou SQLite in-memory ou Testcontainers | InMemory tem caveats |
| Testcontainers Java | Testcontainers.NET | Mesmo container, pacote diferente |

## Build / Project file

| Java | .NET | Notas |
|---|---|---|
| `pom.xml` (Maven) | `.csproj` + `.sln` | XML diferente |
| `build.gradle.kts` | `.csproj` | — |
| `<dependency>` | `<PackageReference>` | NuGet vs Maven Central |
| `mvn clean package` | `dotnet build` + `dotnet publish` | — |
| `mvn test` | `dotnet test` | — |

## Application config

| Java | .NET | Notas |
|---|---|---|
| `application.yml` | `appsettings.json` + `appsettings.{env}.json` | YAML não nativo, usar `Microsoft.Extensions.Configuration.Yaml` se quiser |
| `@ConfigurationProperties("app")` | `services.Configure<AppOptions>(config.GetSection("App"))` | — |
| Profiles (`@Profile("dev")`) | `ASPNETCORE_ENVIRONMENT=Development` + `appsettings.Development.json` | — |
