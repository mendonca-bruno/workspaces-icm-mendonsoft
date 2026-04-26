# Anti-Patterns de Migração

Erros comuns ao traduzir entre Java/Spring, .NET/ASP.NET e Python. Stage 03 (mapping) e stage 05 (translate) devem auditar contra esta lista.

## 1. Tradução literal sem refatoração idiomática

**Sintoma:** o código alvo "funciona" mas é uma cópia palavra-por-palavra do source, sem usar idiomas da linguagem alvo.

**Exemplo ruim** (Java → Python):
```python
class UserService:
    def __init__(self, user_repository):
        self.user_repository = user_repository

    def find_user_by_id(self, id):
        optional_user = self.user_repository.find_by_id(id)
        if optional_user.is_present():
            return optional_user.get()
        else:
            raise UserNotFoundException()
```

**Idiomático:**
```python
class UserService:
    def __init__(self, user_repo: UserRepository):
        self._repo = user_repo

    def get_user(self, user_id: int) -> User:
        if user := self._repo.get(user_id):
            return user
        raise UserNotFound(user_id)
```

**Mitigação:** stage 03 obriga a entrada `idiom_decision` no translation-map por arquivo (não apenas nome de classe).

## 2. Manter modelo de erro do source

**Sintoma:** checked exceptions em Python, ou exceções em vez de Result/Optional onde a linguagem alvo prefere outro modelo.

**Mapping:**
- Java `throws CheckedException` → Python: documentar em docstring + raise; .NET: documentar em XML doc + throw.
- Java `Optional<T>` → .NET: `T?` (nullable reference); Python: `T | None` ou `Optional[T]`.
- Java `try/catch/finally` → Python: `try/except/finally` ou context manager (`with`); .NET: `try/catch/finally` ou `using`.
- .NET `Result<T, E>` (FluentResults) → Java: pode virar exception ou um wrapper (Vavr `Either`); Python: tuple `(value, error)` ou raise.

**Mitigação:** ver `idiom-mapping/error-model.md`.

## 3. Persistência: 1:1 entity mapping

**Sintoma:** copiar `@Entity` Java diretamente para `class Foo(Base)` Python ou `class Foo` C#, sem revisar relacionamentos lazy/eager, cascade, fetch type.

**Armadilhas:**
- Hibernate `@OneToMany(fetch = LAZY)` ↔ EF Core: comportamento diferente; EF Core não tem proxy lazy default desde EF 6.
- `@JoinColumn` JPA ↔ EF Core `[ForeignKey]`: nomes de colunas FK podem divergir.
- SQLAlchemy `lazy='select'` (default) ↔ Hibernate `LAZY`: SQLAlchemy só lazy-loa em sessão ativa.

**Mitigação:** stage 03 marca cada entity com `relationship_strategy` explicit no CSV.

## 4. Concorrência mal traduzida

**Sintoma:** `CompletableFuture` virando `Task` ou `async def` sem revisar event loop, thread pool, blocking calls.

**Armadilhas:**
- Java `@Async` (Spring) → Python `async def`: NÃO é equivalente. Spring `@Async` usa thread pool; Python `async` usa event loop. Bloqueia se misturar.
- .NET `async/await` ↔ Python `asyncio`: parecidos mas Python tem GIL, profile diferente.
- Java threads ↔ Python threads: GIL muda comportamento; CPU-bound em Python ≠ thread, ≠ multiprocessing.

**Mitigação:** `idiom-mapping/concurrency.md` define matriz canônica.

## 5. Refletir injeção de dependência por anotação

**Sintoma:** copiar `@Autowired` para `[Inject]` ou similar sem usar o container nativo da target.

**Mapping correto:**
- Java Spring `@Service` + `@Autowired` ctor → .NET: `public class FooService(IBar bar)` (primary ctor) + `services.AddScoped<IFooService, FooService>()` em `Program.cs`.
- → Python FastAPI: `Depends(get_bar)` no parâmetro do endpoint, ou estrutura mais leve (Protocol + manual instantiation).

**Mitigação:** ver pattern-equivalences específico.

## 6. Configuration sprawl

**Sintoma:** `application.yml` virar 17 arquivos `appsettings.*.json` ou vice-versa sem normalizar.

**Heurística:** 1 arquivo de config por ambiente (dev/staging/prod) é suficiente. Variáveis sensíveis vão para env vars / secret manager.

## 7. Testes traduzidos 1:1 que viram fricção

**Sintoma:** `@SpringBootTest` virando `[Fact] WebApplicationFactory<>` com mesmo escopo, mesmas fixtures, mesma duração — mas o framework alvo tem ferramentas mais leves.

**Mitigação:** stage 06 (validate-equivalence) prioriza **fixtures HTTP/contract compartilhadas** em vez de tradução literal de testes. Ver `equivalence-rubric.md` níveis L3/L4.

## 8. Perder transactionality

**Sintoma:** `@Transactional` Java sumir no target porque o tradutor "esqueceu".

**Equivalentes:**
- .NET: `using var tx = await dbContext.Database.BeginTransactionAsync()` ou `TransactionScope`.
- Python SQLAlchemy: `with session.begin():` ou async equivalent.
- Python Django: `@transaction.atomic` decorator ou context manager.

**Mitigação:** stage 03 marca cada método com flag `transactional` no CSV. Stage 05 falha audit se ausente.

## 9. Misnaming após tradução

**Sintoma:** Java `getUserById` virando Python `get_user_by_id` (correto) mas `UserController` virando `user_controller` (errado em PascalCase em arquivo, mas snake em método). Inverso também: Python `user_id` virando .NET `user_id` em vez de `UserId`.

**Convenções:**
- Java: `PascalCase` classes, `camelCase` métodos/vars, `UPPER_SNAKE` constantes.
- .NET: `PascalCase` classes/métodos públicos/propriedades, `_camelCase` campos privados, `UPPER_SNAKE` const.
- Python: `PascalCase` classes, `snake_case` funções/vars/módulos, `UPPER_SNAKE` constantes.

**Mitigação:** stage 05 aplica renaming canônico per-language como pós-processamento.

## 10. Public surface drift

**Sintoma:** target tem mais ou menos APIs públicas que source. Equivalência impossível de medir.

**Mitigação:** `surface-map.json` (stage 01) lista 100% da public surface; stage 03 obriga 1:1 mapping (cada entrada tem `target_path` ou `action=skip` com justificativa). Stage 06 audita.

## Checklist final (auditoria pré-checkpoint #2)

- [ ] Cada arquivo no CSV tem `idiom_decision` preenchido (não vazio).
- [ ] Anti-pattern 1 (tradução literal): nenhuma classe com `idiom_decision = "literal"` para arquivos não-DTO.
- [ ] Anti-pattern 3 (entities): cada entity tem `relationship_strategy` explícita.
- [ ] Anti-pattern 4 (concurrency): cada arquivo com palavras `async`/`Future`/`Task`/`await` tem `concurrency_strategy`.
- [ ] Anti-pattern 8 (transactional): cada método com `@Transactional` tem flag no CSV.
- [ ] Anti-pattern 10 (surface drift): toda entrada de `surface-map.json` tem destino no CSV.
