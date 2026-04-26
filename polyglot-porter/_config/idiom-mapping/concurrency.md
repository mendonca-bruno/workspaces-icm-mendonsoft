# Idiom Mapping — Concorrência e Async

Concorrência é a área mais propensa a bugs sutis em migração. Cada linguagem tem um runtime diferente: JVM threads + ForkJoinPool, .NET TPL + thread pool, Python event loop + GIL. **Tradução literal causa deadlock ou perda silenciosa de paralelismo.**

## Matriz cross-linguagem

| Conceito | Java | .NET (C#) | Python |
|---|---|---|---|
| Future | `CompletableFuture<T>` | `Task<T>` | `asyncio.Future[T]` ou awaitable |
| Async function | método `CompletableFuture<T>` ou `@Async` | `async Task<T>` | `async def` |
| Await | `cf.get()` (bloqueia) ou `cf.thenApply(...)` | `await cf` | `await coro` |
| Run on pool | `CompletableFuture.supplyAsync(...)` | `Task.Run(...)` | `loop.run_in_executor(...)` |
| Cancellation | `Future.cancel(true)` (best-effort) | `CancellationToken` (cooperative) | `Task.cancel()` (raise CancelledError) |
| Thread (kernel) | `new Thread()` | `Thread` (raramente) | `threading.Thread` (GIL) |
| Process | `ProcessBuilder` | `Process.Start` | `multiprocessing` |
| Lock | `synchronized` ou `ReentrantLock` | `lock` ou `SemaphoreSlim` | `threading.Lock` ou `asyncio.Lock` |
| Channel | `BlockingQueue<T>` | `Channel<T>` | `asyncio.Queue` |
| Reactive | Project Reactor (`Mono`, `Flux`) | `IAsyncEnumerable<T>` | async generators |

## Princípios de tradução

### 1. NÃO traduzir `@Async` (Spring) literalmente para `async def` (Python)

- Java `@Async`: o método roda em **thread pool** separada. O caller pode continuar.
- Python `async def`: o método cria uma **coroutine**. Sem `await` ela não roda. NÃO usa thread pool por default.

**Errado:**
```java
@Async
public CompletableFuture<String> compute() {
    return CompletableFuture.completedFuture(heavyWork());
}
```
→ traduzir para Python como:
```python
async def compute() -> str:
    return heavy_work()  # ROUBADO: roda no event loop, bloqueia tudo
```

**Certo:**
```python
async def compute() -> str:
    loop = asyncio.get_event_loop()
    return await loop.run_in_executor(None, heavy_work)
```

### 2. .NET `Task.Run` ↔ Python `run_in_executor`

`Task.Run` despacha para o thread pool. Python equivalente é `loop.run_in_executor(None, fn, *args)`. NÃO `asyncio.create_task` — esse é para coroutines.

### 3. Cancellation: cooperative vs interruptive

- Java `Thread.interrupt()`: interruptive (mas só se o código checa).
- .NET `CancellationToken`: cooperative (precisa passar adiante e checar).
- Python `asyncio.CancelledError`: cooperative (raised no `await`).

**Regra:** ao traduzir, manter o modelo cooperative se source é .NET ou Python. Se source é Java com `interrupt()`, traduzir para cooperative explícito.

### 4. CPU-bound em Python NÃO pode usar threading

Por causa do GIL, threads Python são úteis apenas para I/O-bound. Para CPU-bound:

- Java threads CPU-bound → Python `multiprocessing.Process` ou `concurrent.futures.ProcessPoolExecutor`.
- .NET threads CPU-bound → idem.

**NUNCA** traduzir `new Thread(() -> heavyMath())` para `threading.Thread(target=heavy_math)` em Python — vai serializar.

### 5. Reactive streams

| Source | Target | Estratégia |
|---|---|---|
| Spring WebFlux `Flux<T>` → .NET | `IAsyncEnumerable<T>` (.NET 8) ou simplesmente `Task<List<T>>` se cardinality permitir | — |
| Spring WebFlux `Flux<T>` → Python | `AsyncIterator[T]` (`async def gen() yield ...`) | — |
| .NET `IAsyncEnumerable<T>` → Java | `Flux<T>` (Reactor) ou `Stream<T>` se collected | — |
| .NET `IAsyncEnumerable<T>` → Python | `AsyncIterator[T]` | Direto |
| Python async generator → Java | `Flux<T>` | — |
| Python async generator → .NET | `IAsyncEnumerable<T>` | Direto |

## Tradução por par

### Java → .NET

| Source | Target | Notas |
|---|---|---|
| `CompletableFuture<T>` | `Task<T>` | Direto |
| `cf.thenApply(f)` | `await cf.ContinueWith(t => f(t.Result))` ou `var x = await cf; var y = f(x);` | Prefere segundo |
| `cf.thenCompose(f)` | `var x = await cf; var y = await f(x);` | — |
| `CompletableFuture.allOf(a, b, c)` | `await Task.WhenAll(a, b, c)` | — |
| `CompletableFuture.anyOf(...)` | `await Task.WhenAny(...)` | — |
| `@Async` (Spring) | `async Task<T>` (já é assíncrono no .NET) | Sem atributo |
| `ExecutorService` | `Channel<T>` ou `Task.Run` | — |
| `synchronized(obj) { ... }` | `lock(obj) { ... }` | — |
| `ReentrantLock` | `lock` ou `SemaphoreSlim` | — |

### Java → Python

| Source | Target | Notas |
|---|---|---|
| `CompletableFuture<T>` | `Coroutine[Any, Any, T]` (return type) | — |
| `cf.get()` (blocking) | `await coro` | — |
| `@Async` em I/O-bound | `async def ...` em FastAPI | OK |
| `@Async` em CPU-bound | `loop.run_in_executor(executor, fn)` com `ProcessPoolExecutor` | Cuidado GIL |
| `synchronized` | `asyncio.Lock` (se async) ou `threading.Lock` (se sync) | — |
| `BlockingQueue<T>` | `asyncio.Queue` ou `queue.Queue` | — |

### .NET → Java

| Source | Target | Notas |
|---|---|---|
| `Task<T>` | `CompletableFuture<T>` | — |
| `await x` | `x.get()` (blocking) ou `x.thenApply(...)` (chain) | Prefere chain |
| `Task.Run(fn)` | `CompletableFuture.supplyAsync(fn)` | — |
| `Task.WhenAll(a, b)` | `CompletableFuture.allOf(a, b).join()` | — |
| `CancellationToken` | parâmetro adicional + `Thread.interrupted()` checks (verboso) | Java não tem direto |
| `Channel<T>` | `BlockingQueue<T>` | — |

### .NET → Python

| Source | Target | Notas |
|---|---|---|
| `Task<T>` | `Coroutine[..., T]` | — |
| `await x` | `await x` | Direto |
| `Task.Run(fn)` | I/O-bound: `asyncio.create_task(fn())`. CPU: `loop.run_in_executor(...)` | — |
| `Task.WhenAll(a, b)` | `asyncio.gather(a, b)` | — |
| `CancellationToken` | `asyncio.Task.cancel()` + try/except `CancelledError` | — |

### Python → Java

| Source | Target | Notas |
|---|---|---|
| `async def f() -> T` | `CompletableFuture<T> f()` ou `Mono<T>` | — |
| `await x` | `x.get()` ou chain | — |
| `asyncio.gather(...)` | `CompletableFuture.allOf(...)` | — |
| `asyncio.create_task(coro)` | `CompletableFuture.runAsync(...)` | — |
| `multiprocessing.Pool` | `ExecutorService` (sem GIL Java) | — |

### Python → .NET

| Source | Target | Notas |
|---|---|---|
| `async def f() -> T` | `async Task<T> F()` | — |
| `await x` | `await x` | — |
| `asyncio.gather(a, b)` | `await Task.WhenAll(a, b)` | — |
| `multiprocessing.Pool` | `Parallel.ForEach` ou `Task.Run` (sem GIL .NET) | — |

## Checklist de auditoria (stage 03)

Para cada arquivo no CSV com indicador de concorrência:

- [ ] `concurrency_strategy` preenchido (`async-io`, `cpu-bound`, `thread-pool`, `synchronous`).
- [ ] Se `cpu-bound` e target=Python: marcar `executor=process_pool`.
- [ ] Se source tem `@Async` Spring: estratégia explícita decidida (não default async/await).
- [ ] Se source tem `CancellationToken` ou Future cancel: política de cancellation traduzida (cooperative em .NET/Python; documentada em Java).
- [ ] Locks: tipo de lock target compatível com modo (sync vs async).
