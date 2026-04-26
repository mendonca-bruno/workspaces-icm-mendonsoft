# Idiom Mapping — Null, Optional, Nullable

Cada linguagem MVP trata "ausência de valor" de forma diferente. Tradução literal causa bugs sutis. Esta matriz é a referência canônica.

## Matriz cross-linguagem

| Conceito | Java | .NET (C#) | Python |
|---|---|---|---|
| "Pode estar ausente" no tipo | `Optional<T>` (preferido) ou `T` (legado, pode ser null) | `T?` (nullable reference type, NRT) | `T \| None` ou `Optional[T]` (typing) |
| Default ausência | `Optional.empty()` | `null` | `None` |
| Construir presença | `Optional.of(x)` | `x` direto | `x` direto |
| Construir ausência tolerante a null | `Optional.ofNullable(x)` | `x` (NRT permite) | `x` (None permitido) |
| Acesso seguro | `opt.map(f).orElse(default)` | `x?.Foo ?? default` | `x.foo if x else default` ou `(x or default).foo` (cuidado com falsy) |
| Throw se ausente | `opt.orElseThrow(() -> ex)` | `x ?? throw new Exception()` | `if x is None: raise ...` |
| Default lazy | `opt.orElseGet(() -> compute())` | `x ?? Compute()` | `x if x is not None else compute()` |

## Regras de tradução

### Java → .NET

| Source pattern | Target pattern |
|---|---|
| `Optional<T> getFoo()` | `T? GetFoo()` (com `<Nullable>enable</Nullable>` no .csproj) |
| `Optional<T> ofNullable(x)` | `x` (deixar fluir; NRT entende) |
| `opt.map(f).orElse(d)` | `x?.Apply(f) ?? d` ou expression-bodied |
| `opt.orElseThrow()` | `x ?? throw new InvalidOperationException()` |
| Field `private String name;` (pode ser null) | `private string? Name { get; init; }` ou `private readonly string? _name;` |

### Java → Python

| Source pattern | Target pattern |
|---|---|
| `Optional<T> getFoo()` | `def get_foo() -> T \| None:` |
| `opt.isPresent()` | `x is not None` (não usar `if x:` — falsy bugs) |
| `opt.map(f).orElse(d)` | `f(x) if x is not None else d` |
| `opt.orElseThrow(() -> ex)` | `if x is None: raise ex` |
| `opt.ifPresent(consumer)` | `if x is not None: consumer(x)` |

### .NET → Java

| Source pattern | Target pattern |
|---|---|
| `T?` (nullable) | `Optional<T>` (preferir) |
| `x?.Foo ?? d` | `Optional.ofNullable(x).map(X::getFoo).orElse(d)` |
| `x ?? throw new ...` | `Optional.ofNullable(x).orElseThrow(() -> new ...)` |
| `string.IsNullOrEmpty(s)` | `s == null \|\| s.isEmpty()` ou `Strings.isNullOrEmpty(s)` (Guava) |

### .NET → Python

| Source pattern | Target pattern |
|---|---|
| `T?` | `T \| None` |
| `x?.Foo ?? d` | `getattr(x, 'foo', d) if x else d` ou checagem explícita |
| `string.IsNullOrEmpty(s)` | `not s` (cuidado com falsy) ou `s is None or s == ""` |

### Python → Java

| Source pattern | Target pattern |
|---|---|
| `T \| None` no return | `Optional<T>` |
| `if x is not None: ...` | `if (x != null) { ... }` ou `Optional.ofNullable(x).ifPresent(...)` |
| `(x or default)` | **CUIDADO**: Java não tem essa coisa de "falsy". Traduzir com checagem explícita. |
| `x.foo if x else None` | `Optional.ofNullable(x).map(X::getFoo).orElse(null)` |

### Python → .NET

| Source pattern | Target pattern |
|---|---|
| `T \| None` | `T?` (NRT) |
| `if x is not None:` | `if (x is not null)` |
| `(x or default)` | **CUIDADO**: traduzir com checagem explícita. C# não trata 0/""/[] como falsy. |

## Anti-pattern: traduzir "falsy" entre linguagens

Python: `if value:` é true para `None`, `0`, `""`, `[]`, `{}`, `False`.
Java/C#: só `null` é falsy (e em C# antigo, `0` em booleano).

Tradução literal de `if x:` para `if (x != null)` muda comportamento se `x` for `0`, `""`, etc.

**Regra:** sempre traduzir Python falsy checks com checagem explícita do tipo:
- `if x:` em string → `if (x != null && !x.isEmpty())`
- `if x:` em number → `if (x != null && x != 0)` (mas verificar intent)
- `if x:` em list → `if (x != null && !x.isEmpty())`

## Quando NÃO usar Optional em Java

- Em fields de entidades JPA: usar `null` simples ou tipos nullable. JPA não suporta bem `Optional` em fields.
- Como parâmetro de método: anti-pattern segundo guias oficiais. Use overloads ou `null` documentado.
- Em coleções: prefira coleção vazia em vez de `Optional<List<T>>`.

## Quando NÃO usar `T?` em C#

- Em DTOs onde o contrato exige presença: declarar `required` (C# 11+) ou throw em construtor.
- Em entidades EF Core: configurar `IsRequired()` no Fluent API ou via `[Required]`.
