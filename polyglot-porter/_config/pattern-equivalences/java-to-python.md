# Pattern Equivalences — Java/Spring Boot → Python (FastAPI default)

Mapeamento canônico. Default target: **FastAPI + SQLAlchemy 2 + Pydantic**. Para Django, ver seção final.

## Web layer (FastAPI)

| Java/Spring | FastAPI equivalente | Notas |
|---|---|---|
| `@RestController` + `@RequestMapping("/api")` | `router = APIRouter(prefix="/api")` | Sem classe wrapper |
| `@GetMapping("/{id}")` | `@router.get("/{id}")` | — |
| `@PostMapping` + `@RequestBody Foo dto` | `@router.post("/")` + parâmetro `dto: FooSchema` | Pydantic auto-deserialize |
| `@PathVariable` | parâmetro do path: `id: int` | Type hint = source |
| `@RequestParam` | parâmetro normal: `q: str = ""` | — |
| `@RequestHeader` | `Header(...)` | — |
| `ResponseEntity<T>` | retornar `T` ou `JSONResponse(content, status_code)` | — |
| `@ControllerAdvice` + `@ExceptionHandler` | `@app.exception_handler(MyException)` | — |
| `@Valid` + bean validation | Pydantic auto-valida via type hints | — |

## DI / lifecycle (FastAPI)

| Java/Spring | FastAPI equivalente | Notas |
|---|---|---|
| `@Service` | classe normal + `Depends(get_service)` factory | Sem container global |
| `@Component` | função/classe + `Depends()` | — |
| `@Repository` | classe + `Depends(get_repo)` | Mesmo padrão |
| `@Configuration` + `@Bean` | função `get_x()` + `Depends(get_x)` | Lazy via `Depends` |
| `@Autowired` (ctor) | parâmetro de função com `Depends()` | Não há "container" |
| `@Qualifier("name")` | múltiplas funções getter (`get_a`, `get_b`) | Naming explícito |
| `@Value("${app.foo}")` | `Settings(BaseSettings)` (pydantic-settings) | — |

## Persistência (JPA → SQLAlchemy 2)

| JPA | SQLAlchemy 2 | Notas |
|---|---|---|
| `@Entity` | `class Foo(Base):` (DeclarativeBase) | — |
| `@Id` + `@GeneratedValue(IDENTITY)` | `id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)` | — |
| `@Column(name = "x")` | `mapped_column("x", String, nullable=False)` | — |
| `@OneToMany(mappedBy = "x", cascade = ALL)` | `items: Mapped[list["Item"]] = relationship(back_populates="parent", cascade="all, delete-orphan")` | — |
| `@ManyToOne` | `parent: Mapped["Parent"] = relationship(back_populates="items")` | — |
| `@JoinColumn(name = "x_id")` | `parent_id: Mapped[int] = mapped_column(ForeignKey("parents.id"))` | — |
| `JpaRepository<T, Long>` | classe Repository manual com `Session` (sem CRUD pronto) | OU usar `sqlmodel`/`piccolo` |
| `@Query("JPQL")` | `select(Foo).where(...)` (SQLAlchemy core) | — |
| `@Transactional` | `with session.begin():` ou `async with session.begin():` | — |
| Flyway migrations | Alembic (`alembic revision --autogenerate`) | — |

## Persistência (Django alternativa)

| JPA | Django ORM | Notas |
|---|---|---|
| `@Entity` | `class Foo(models.Model):` | App-scoped |
| `@Id` (auto) | implicit `id = AutoField` | — |
| `@OneToMany` | `ForeignKey` no lado many; reverse via `_set` | Inverso da relação |
| `@Transactional` | `@transaction.atomic` | — |
| Flyway | Django migrations (`makemigrations` + `migrate`) | — |

## Async / Concurrency

| Java | Python | Notas |
|---|---|---|
| `CompletableFuture<T>` | `asyncio.Future[T]` ou `Coroutine[T]` | Direto-ish |
| `@Async` Spring | `async def` em rota FastAPI | NÃO equivalentes; ver `idiom-mapping/concurrency.md` |
| `Mono<T>` / `Flux<T>` | `AsyncIterator[T]` ou `async generator` | Streaming |
| `ExecutorService` | `concurrent.futures.ThreadPoolExecutor` ou `asyncio.create_task()` | GIL muda comportamento |
| Threads (CPU-bound) | `multiprocessing` | NÃO `threading` (GIL) |

## Validation

| Java | Python (Pydantic) | Notas |
|---|---|---|
| `@NotNull` | tipo sem `Optional[]` | Pydantic exige se não tem default |
| `@Size(min, max)` | `Field(min_length=, max_length=)` | — |
| `@Email` | `EmailStr` (extra do Pydantic) | `pip install pydantic[email]` |
| `@Pattern(regex)` | `Field(pattern=r"...")` | — |
| `@Min/@Max` | `Field(ge=, le=)` | — |
| Custom `ConstraintValidator` | `@field_validator` ou `@model_validator` | — |

## Logging

| Java | Python | Notas |
|---|---|---|
| SLF4J `LoggerFactory.getLogger(Foo.class)` | `logging.getLogger(__name__)` | — |
| `log.info("msg {}", arg)` | `logger.info("msg %s", arg)` ou `logger.info("msg", extra={...})` | Use `structlog` para structured |

## Testing

| Java | Python | Notas |
|---|---|---|
| JUnit 5 `@Test` | função `def test_...():` com pytest | — |
| `@ParameterizedTest` | `@pytest.mark.parametrize(...)` | — |
| Mockito `mock()` | `unittest.mock.Mock()` ou `pytest-mock` | — |
| `@SpringBootTest` | `TestClient(app)` (FastAPI) ou `pytest-django` | — |
| Testcontainers Java | Testcontainers Python | Mesmo container |

## Build / Project file

| Java | Python | Notas |
|---|---|---|
| `pom.xml` | `pyproject.toml` | PEP 621 |
| `<dependency>` | `[project.dependencies]` ou `requirements.txt` | — |
| `mvn package` | `python -m build` | — |
| `mvn test` | `pytest` | — |

## Application config

| Java | Python | Notas |
|---|---|---|
| `application.yml` | `.env` + `Settings(BaseSettings)` | pydantic-settings |
| `@ConfigurationProperties("app")` | `class AppSettings(BaseSettings): model_config = SettingsConfigDict(env_prefix="APP_")` | — |
| `@Profile("dev")` | env var `ENV=dev` + lógica em `Settings` | — |
