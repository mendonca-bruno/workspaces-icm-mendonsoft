# Idiom Mapping — Modelo de Erros

Java tem checked + unchecked exceptions. C# só unchecked. Python só unchecked + alguns Result libs. Este documento define a tradução canônica.

## Matriz cross-linguagem

| Conceito | Java | .NET (C#) | Python |
|---|---|---|---|
| Erro recuperável | Checked exception (`extends Exception`) | Exception (não há "checked") | Exception subclass |
| Erro irrecuperável | RuntimeException (`extends RuntimeException`) | Exception (mesmo) | Exception |
| Throw | `throw new Foo()` | `throw new Foo()` | `raise Foo()` |
| Catch | `try { ... } catch (Foo e) { ... }` | `try { ... } catch (Foo e) { ... }` | `try: ... except Foo as e: ...` |
| Re-throw | `throw new Bar(e)` ou `throw e;` | `throw;` (preserva stack) ou `throw new Bar(e)` | `raise Bar() from e` |
| Finally | `finally { ... }` | `finally { ... }` ou `using` | `finally:` ou `with` |
| Either / Result | `Either<L, R>` (Vavr) ou `Result<T>` | FluentResults / OneOf | `(value, error)` tuple ou `Result` lib |

## Tradução: Java → .NET

| Java | .NET | Notas |
|---|---|---|
| `void foo() throws IOException` | `void Foo()` (sem `throws`) | Documentar em XML doc `<exception>` |
| `try { ... } catch (IOException e) { log.error("...", e); }` | `try { ... } catch (IOException e) { _logger.LogError(e, "..."); }` | — |
| Custom checked: `class FooException extends Exception` | `class FooException : Exception` | Sem distinção checked |
| `try-with-resources (var x = open())` | `using var x = Open();` | Idiomático |
| `Optional<T>` para erro recuperável | `T?` ou `OneOf<T, Error>` | Decidir caso a caso |

**Regra:** se source usa checked exception como "Result tipado", considere migrar para `OneOf<T, Error>` ou similar em .NET. Se usa só para sinalizar erro, `Exception` simples é suficiente.

## Tradução: Java → Python

| Java | Python | Notas |
|---|---|---|
| `void foo() throws IOException` | `def foo() -> None:` | Documentar em docstring `Raises:` |
| `try { ... } catch (IOException e) { ... }` | `try: ...\n except IOError as e: ...` | — |
| `class FooException extends RuntimeException` | `class FooError(Exception):` | Convenção: nome termina com `Error` |
| `finally { closeStream(); }` | `with open(...) as f:` (preferido) ou `try/finally` | Context manager idiomático |
| `try-with-resources` | `with x:` | — |

**Regra:** Python prefere context managers (`with`) para cleanup. Não traduza `try-with-resources` literalmente para `try/finally` se o recurso é um context manager.

## Tradução: .NET → Java

| .NET | Java | Notas |
|---|---|---|
| `throw new InvalidOperationException()` | `throw new IllegalStateException()` | Mapeamento canônico |
| `throw new ArgumentException()` | `throw new IllegalArgumentException()` | — |
| `throw new ArgumentNullException(nameof(x))` | `Objects.requireNonNull(x, "x")` ou `throw new NullPointerException("x")` | — |
| `using` | `try-with-resources` | — |
| FluentResults `Result<T>` | `Either<Error, T>` (Vavr) ou checked exception | Decidir |

## Tradução: .NET → Python

| .NET | Python | Notas |
|---|---|---|
| `throw new InvalidOperationException()` | `raise RuntimeError()` ou custom | — |
| `throw new ArgumentException("x")` | `raise ValueError("x")` | — |
| `throw new ArgumentNullException()` | `raise TypeError()` ou `raise ValueError()` | Python não distingue tão fino |
| `using` | `with` | — |
| `try { ... } catch when (cond)` | `try: ... except E as e:\n if not cond: raise` | Python não tem filter na cláusula |

## Tradução: Python → Java

| Python | Java | Notas |
|---|---|---|
| `raise ValueError("x")` | `throw new IllegalArgumentException("x")` | — |
| `raise RuntimeError()` | `throw new RuntimeException()` | — |
| `raise NotImplementedError()` | `throw new UnsupportedOperationException()` | — |
| `raise FooError() from e` | `throw new FooException(e)` | — |
| `with x:` | `try-with-resources (x)` | Se `x` é AutoCloseable |
| `try: ... except: ...` (bare) | **NUNCA** traduzir literal — exigir tipo específico | bare except é anti-pattern |

## Tradução: Python → .NET

| Python | .NET | Notas |
|---|---|---|
| `raise ValueError("x")` | `throw new ArgumentException("x")` | — |
| `raise RuntimeError()` | `throw new InvalidOperationException()` | — |
| `raise FooError() from e` | `throw new FooException(e.Message, e)` | Inner exception |
| `with x:` | `using var ... = x;` | — |

## Anti-patterns

1. **Catch and swallow.** `catch (Exception e) {}` → idem em qualquer linguagem é bug. Sempre logar OU re-throw.
2. **Catch base class.** `catch (Exception e)` em Java/C# ou `except Exception as e` em Python perde contexto. Prefira tipo específico.
3. **Lose the cause.** Re-throw sem encadear (`throw new Bar()` em vez de `throw new Bar(e)`) perde stack.
4. **Translate checked → unchecked silenciosamente.** Java→C#/Python: documentar em XML doc / docstring que pode lançar; não esconder.
5. **Map todas as exceptions para 500 no controller.** Use `@ControllerAdvice` / `IExceptionHandler` / `@app.exception_handler` com mapeamento explícito por tipo.

## Mapping para HTTP status (todos os pares)

| Tipo de erro lógico | HTTP | Java | .NET | Python (FastAPI) |
|---|---|---|---|---|
| Validation falhou | 400 | `MethodArgumentNotValidException` (auto) | `[ApiController]` auto-400 | `RequestValidationError` (auto) |
| Não encontrado | 404 | custom `NotFoundException` mapeado | `NotFound()` ou throw → handler | `HTTPException(404)` |
| Não autorizado | 401 | Spring Security | ASP.NET Auth middleware | dep `Depends(get_user)` raise |
| Forbidden | 403 | `@PreAuthorize` falha | `[Authorize]` falha | dep raise 403 |
| Conflito (ex: dup key) | 409 | custom mapeado | custom mapeado | `HTTPException(409)` |
| Server error | 500 | unhandled → handler default | unhandled → middleware default | unhandled → 500 default |
