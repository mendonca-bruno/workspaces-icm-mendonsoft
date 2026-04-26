# Pattern Equivalences — Python (FastAPI/Django) → Java/Spring Boot

Mapeamento canônico. Inverso de [`java-to-python.md`](java-to-python.md). Default target: **Spring Boot 3 + Spring Web + Spring Data JPA**.

## Web layer (FastAPI → Spring)

| FastAPI | Java/Spring equivalente | Notas |
|---|---|---|
| `router = APIRouter(prefix="/api")` | `@RestController` + `@RequestMapping("/api")` em classe | Spring é class-based |
| `@router.get("/{id}")` | `@GetMapping("/{id}")` em método | — |
| `@router.post("/")` + `dto: FooSchema` | `@PostMapping` + `@RequestBody @Valid FooDto dto` | Pydantic ↔ Bean Validation |
| path param `id: int` | `@PathVariable Long id` | — |
| query param `q: str = ""` | `@RequestParam(defaultValue = "") String q` | — |
| `Header(...)` | `@RequestHeader` | — |
| return `T` | return `ResponseEntity.ok(t)` ou `T` direto | — |
| `@app.exception_handler(MyException)` | `@ControllerAdvice` + `@ExceptionHandler` | — |

## Web layer (Django REST Framework → Spring)

| DRF | Spring | Notas |
|---|---|---|
| `class FooViewSet(ModelViewSet)` | `@RestController` + métodos CRUD manuais | Spring não tem ViewSet |
| `serializer_class = FooSerializer` | `@RequestBody FooDto` + manual mapping ou MapStruct | — |
| `permission_classes = [IsAuthenticated]` | Spring Security + `@PreAuthorize("isAuthenticated()")` | — |
| `urls.py` `router.register()` | `@RequestMapping` na classe + `@GetMapping`/etc. nos métodos | — |

## DI / lifecycle

| Python | Java/Spring | Notas |
|---|---|---|
| `Depends(get_service)` no parâmetro | `@Autowired` ctor injection | Spring tem container |
| classe sem decorator + factory `get_x()` | `@Service` na classe | — |
| `Settings(BaseSettings)` (pydantic-settings) | `@ConfigurationProperties("app")` em classe | — |
| factory `get_db()` | `@Configuration` + `@Bean DataSource` | — |

## Persistência (SQLAlchemy 2 → JPA)

| SQLAlchemy 2 | JPA | Notas |
|---|---|---|
| `class Foo(Base):` | `@Entity public class Foo` | — |
| `id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)` | `@Id @GeneratedValue(strategy = IDENTITY) private Long id;` | — |
| `mapped_column("x", String)` | `@Column(name = "x") private String x;` | — |
| `relationship(back_populates="parent", cascade="all, delete-orphan")` | `@OneToMany(mappedBy = "parent", cascade = ALL, orphanRemoval = true)` | — |
| `parent_id: Mapped[int] = mapped_column(ForeignKey("parents.id"))` | `@ManyToOne @JoinColumn(name = "parent_id") private Parent parent;` | — |
| Repository manual com `Session` | `interface FooRepository extends JpaRepository<Foo, Long>` | Spring Data dá CRUD |
| `select(Foo).where(...)` | JPQL `@Query` ou `Specification<Foo>` | — |
| `with session.begin():` | `@Transactional` no método | — |
| Alembic migrations | Flyway (`db/migration/V1__init.sql`) | — |

## Persistência (Django ORM → JPA)

| Django ORM | JPA | Notas |
|---|---|---|
| `class Foo(models.Model):` | `@Entity public class Foo` | — |
| `name = models.CharField(max_length=100)` | `@Column(length = 100) private String name;` | — |
| `ForeignKey('Bar', on_delete=CASCADE)` | `@ManyToOne @JoinColumn` + cascade JPA | — |
| `objects.filter(...)` (manager) | `Specification` ou query method `findByX(...)` | — |
| `@transaction.atomic` | `@Transactional` | — |
| Django migrations | Flyway | — |

## Async / Concurrency

| Python | Java | Notas |
|---|---|---|
| `async def` em FastAPI | método retornando `CompletableFuture<T>` ou `Mono<T>` (WebFlux) | NÃO equivalente direto; ver `idiom-mapping/concurrency.md` |
| `asyncio.gather(...)` | `CompletableFuture.allOf(...)` | — |
| `ThreadPoolExecutor` | `ExecutorService` (`Executors.newFixedThreadPool()`) | — |
| `multiprocessing` (CPU-bound) | Threads (sem GIL) ou `parallelStream()` | — |

## Validation

| Python (Pydantic) | Java | Notas |
|---|---|---|
| tipo sem `Optional[]` | `@NotNull` | Java requer anotação explícita |
| `Field(min_length=, max_length=)` | `@Size(min, max)` | — |
| `EmailStr` | `@Email` | — |
| `Field(pattern=r"...")` | `@Pattern(regex)` | — |
| `Field(ge=, le=)` | `@Min(min)` + `@Max(max)` | — |
| `@field_validator` | `ConstraintValidator` custom | — |

## Logging

| Python | Java | Notas |
|---|---|---|
| `logging.getLogger(__name__)` | `LoggerFactory.getLogger(Foo.class)` (SLF4J) | — |
| `logger.info("msg %s", arg)` | `log.info("msg {}", arg)` | — |
| `structlog` | Logback structured + JSON encoder | — |

## Testing

| Python | Java | Notas |
|---|---|---|
| `def test_...():` (pytest) | `@Test` (JUnit 5) | — |
| `@pytest.mark.parametrize(...)` | `@ParameterizedTest` + `@ValueSource` | — |
| `unittest.mock.Mock()` | Mockito `mock()` | — |
| `TestClient(app)` (FastAPI) | `@SpringBootTest` + `MockMvc` ou `TestRestTemplate` | — |
| Testcontainers Python | Testcontainers Java | — |

## Build / Project file

| Python | Java | Notas |
|---|---|---|
| `pyproject.toml` | `pom.xml` ou `build.gradle.kts` | — |
| `[project.dependencies]` | `<dependency>` Maven | — |
| `python -m build` | `mvn package` | — |
| `pytest` | `mvn test` | — |

## Application config

| Python | Java | Notas |
|---|---|---|
| `.env` + `Settings(BaseSettings)` | `application.yml` + `@ConfigurationProperties` | — |
| `SettingsConfigDict(env_prefix="APP_")` | `@ConfigurationProperties(prefix = "app")` | — |
| `ENV=dev` env var | `@Profile("dev")` + `application-dev.yml` | — |
