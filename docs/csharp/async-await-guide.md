---
layout: page
title: C# - Async/Await Guide
parent: C#
---

# Async/Await in .NET — From Fundamentals to Expert

> `async`/`await` makes asynchronous code read like synchronous code. That readability hides a lot of machinery, and the gap between "looks simple" and "behaves correctly" is where most real-world bugs live — deadlocks, thread starvation, swallowed exceptions and subtle context capture. This guide walks from the basics all the way to expert-level pitfalls, with a strong focus on the classic **sync-over-async deadlock** and the best practices that keep libraries and UI code safe.

---

## Table of Contents

1. [Why Asynchrony?](#why-asynchrony)
2. [Mental Model: What `await` Actually Does](#mental-model-what-await-actually-does)
3. [The Basics](#the-basics)
4. [Tasks, `Task<T>` and `ValueTask`](#tasks-taskt-and-valuetask)
5. [The `SynchronizationContext`](#the-synchronizationcontext)
6. [The Sync-over-Async Deadlock](#the-sync-over-async-deadlock)
7. [`ConfigureAwait(false)` — The Fix and Its Fine Print](#configureawaitfalse--the-fix-and-its-fine-print)
8. [Rules of Thumb: Library vs. UI Code](#rules-of-thumb-library-vs-ui-code)
9. [Async All the Way](#async-all-the-way)
10. [Cancellation](#cancellation)
11. [Exceptions in Async Code](#exceptions-in-async-code)
12. [`async void` — When and Why to Avoid It](#async-void--when-and-why-to-avoid-it)
13. [Concurrency: Running Work in Parallel](#concurrency-running-work-in-parallel)
14. [Expert Pitfalls](#expert-pitfalls)
15. [A Realistic Migration Strategy](#a-realistic-migration-strategy)
16. [Guarding Against Regressions](#guarding-against-regressions)
17. [Diagnosing a Frozen App](#diagnosing-a-frozen-app)
18. [Checklist](#checklist)
19. [Further Reading](#further-reading)

---

## Why Asynchrony?

Asynchronous code is about **not blocking a thread while waiting** for something that is inherently slow: network calls, disk I/O, database queries, timers. There are two distinct motivations:

- **Scalability (server side):** A blocked thread consumes memory (~1 MB stack) and a thread-pool slot while doing nothing. Under load, blocking on I/O leads to thread-pool starvation and collapsing throughput. Async frees the thread to serve other requests while the I/O is in flight.
- **Responsiveness (client side):** UI frameworks (WPF, WinForms, MAUI, Avalonia) run all UI updates on a single **UI thread**. If you block it, the window freezes — no repaint, no input. Async keeps the UI thread free.

> **Async is not parallelism.** `await` does not create a thread. Truly CPU-bound work still needs a thread (e.g. `Task.Run`). Async is primarily about *waiting efficiently*, not *computing faster*.

---

## Mental Model: What `await` Actually Does

When the compiler sees an `async` method, it rewrites it into a **state machine**. Each `await` becomes a potential suspension point:

1. Evaluate the awaited expression (usually a `Task`).
2. If it is **already completed**, continue synchronously — no suspension, no context switch.
3. If it is **not** completed, register a *continuation* (the code after the `await`), then **return** to the caller. The current thread is released.
4. When the awaited operation completes, the continuation is scheduled to run — by default, back on the **captured context** (see below).

The crucial insight: `await` **returns control to the caller**. It does not "wait" by blocking a thread. The method's remaining body is packaged up and resumed later.

---

## The Basics

```csharp
public async Task<string> GetGreetingAsync()
{
    // The HTTP call is I/O — the thread is released while waiting.
    using var client = new HttpClient();
    string name = await client.GetStringAsync("https://api.example.com/name");
    return $"Hello, {name}!";
}
```

Rules to internalize early:

- An `async` method should return `Task`, `Task<T>`, or `ValueTask`/`ValueTask<T>` — **never `void`**, except for event handlers.
- The `async` keyword only *enables* `await`; it does not by itself make anything run on another thread.
- Suffix async methods with `Async` by convention (`ReadAsync`, `SaveChangesAsync`).
- If a method has nothing to await, don't mark it `async` — return the `Task` directly or use `Task.FromResult`.

```csharp
// No await needed — just return the task (avoids state-machine overhead).
public Task<int> GetCachedValueAsync() => Task.FromResult(_cachedValue);
```

---

## Tasks, `Task<T>` and `ValueTask`

- **`Task`** — represents an asynchronous operation with no result.
- **`Task<T>`** — an asynchronous operation that produces a value of type `T`.
- **`ValueTask` / `ValueTask<T>`** — a struct-based alternative that avoids a heap allocation when the result is *often available synchronously* (e.g. a cache hit). Use it in hot paths, but respect its rules:
  - Never `await` a `ValueTask` more than once.
  - Never block on it, never access `.Result` before completion, never `await` it concurrently.
  - When in doubt, use `Task<T>` — it is more forgiving.

```csharp
public ValueTask<int> GetAsync(string key)
{
    if (_cache.TryGetValue(key, out int value))
        return new ValueTask<int>(value);       // synchronous fast path, no allocation

    return new ValueTask<int>(LoadFromDbAsync(key)); // async slow path
}
```

---

## The `SynchronizationContext`

A `SynchronizationContext` is an abstraction for "run this delegate on the *right* thread/queue." Different environments install different ones:

| Environment | Context behavior |
|-------------|------------------|
| WPF / WinForms / MAUI | Marshals continuations back onto the **single UI thread** |
| ASP.NET Core | **No** `SynchronizationContext` — continuations run on any thread-pool thread |
| Classic ASP.NET (.NET Framework) | Request-bound context (a common deadlock source) |
| Console app / thread-pool thread | Usually no context |

By default, `await` **captures the current `SynchronizationContext`** (or `TaskScheduler`) and posts the continuation back to it. This is what lets you safely write:

```csharp
private async void LoadButton_Click(object sender, RoutedEventArgs e)
{
    var data = await _service.LoadAsync();  // UI thread released here
    ResultLabel.Content = data;             // resumes on the UI thread — safe!
}
```

The continuation `ResultLabel.Content = data;` runs back on the UI thread precisely *because* the context was captured. This is a feature — right up until it causes a deadlock.

---

## The Sync-over-Async Deadlock

This is the single most infamous async bug in .NET. It appears when **UI (or classic ASP.NET) code calls an async method synchronously**, blocking on the result via `.Result`, `.Wait()` or `.GetAwaiter().GetResult()`.

```csharp
// ❌ DEADLOCK on a UI thread or classic ASP.NET request thread
private void LoadButton_Click(object sender, RoutedEventArgs e)
{
    // Blocks the UI thread waiting for the task...
    var data = _service.LoadAsync().Result;
    ResultLabel.Content = data;
}

// Inside the library:
public async Task<string> LoadAsync()
{
    var response = await _http.GetStringAsync(_url); // captures the UI context
    return Process(response);                         // wants to resume on the UI thread
}
```

### Why it locks up — step by step

1. The UI thread calls `.Result` and **blocks**, waiting for `LoadAsync` to finish.
2. Inside `LoadAsync`, the `await` captured the UI `SynchronizationContext`. When the HTTP call finishes, the continuation (`return Process(response);`) is **posted back to the UI thread**.
3. But the UI thread is **blocked** on `.Result` — it cannot pump its message queue, so it never picks up the continuation.
4. The continuation can't run, so the task never completes; the `.Result` call never returns. **Both wait on each other forever.** The app freezes.

The trap is that it *looks* like a network hang, but the network call already succeeded — the deadlock is purely about the continuation never getting a free UI thread.

---

## `ConfigureAwait(false)` — The Fix and Its Fine Print

`ConfigureAwait(false)` tells a specific `await`: **do not capture the context; resume the continuation on any available thread-pool thread.**

```csharp
public async Task<string> LoadAsync()
{
    // Continuation no longer needs the UI thread → no deadlock even if the
    // caller blocks on .Result.
    var response = await _http.GetStringAsync(_url).ConfigureAwait(false);
    return Process(response);
}
```

Now step 2 above changes: the continuation runs on a thread-pool thread, never touching the blocked UI thread. The task completes, `.Result` returns, deadlock avoided.

### The fine print that trips everyone up

- **`ConfigureAwait(false)` is per-`await`, not recursive.** It only affects the `await` it is attached to. Every subsequent `await` in the same method needs its own `ConfigureAwait(false)`.
- **It must live on the *inner* awaits (inside the library), not at the call site.** A `ConfigureAwait(false)` written by the *consumer* does **not** fix a deadlock caused deep inside a library — the problematic `await` is the library's.
- **One missed `await` reintroduces the deadlock.** The convention has to be applied *without gaps*. A single un-configured `await` anywhere on the path back to the blocking caller is enough to reproduce the freeze.
- **After `ConfigureAwait(false)`, you are no longer on the UI thread.** Do not touch UI elements after such an await in UI code — it will throw an invalid-cross-thread exception.
- **In ASP.NET Core it is largely a no-op** (there is no `SynchronizationContext`), but it remains harmless and is still recommended in shared libraries that might also run under WPF/WinForms.

---

## Rules of Thumb: Library vs. UI Code

| Context | Recommendation |
|---------|----------------|
| **Library / infrastructure code** (never touches UI) | **Always** `ConfigureAwait(false)` on every `await` |
| **UI / application code** (wants to update the UI after `await`) | **No** `ConfigureAwait(false)` — returning to the UI thread is exactly what you want |

The reasoning: a reusable library cannot know whether its caller has a `SynchronizationContext` or is blocking on it. By never capturing the context, the library becomes **deadlock-safe regardless of how it is consumed** — even by legacy code that still blocks synchronously. Application/UI code, by contrast, *benefits* from the capture so that post-`await` code can safely touch controls.

```csharp
// Library method — deadlock-safe for any caller
public async Task<Order> GetOrderAsync(int id)
{
    var dto = await _db.QueryAsync(id).ConfigureAwait(false);
    var enriched = await _pricing.EnrichAsync(dto).ConfigureAwait(false);
    return Map(enriched);
}

// UI handler — keep the context so we can update controls
private async void Refresh_Click(object sender, RoutedEventArgs e)
{
    var order = await _service.GetOrderAsync(42);  // no ConfigureAwait here
    OrderView.DataContext = order;                  // back on UI thread — safe
}
```

---

## Async All the Way

The cleanest long-term solution is to **never block on async in the first place**. Keep the `async`/`await` chain unbroken from the lowest I/O call all the way up to the entry point:

- Library → service → view model → event handler, each `await`-ing the next.
- The only place `async void` is acceptable is the top-level UI event handler.
- Console apps can use `async Task Main`.

```csharp
// async all the way — no .Result, no .Wait(), no deadlock possible
private async void Save_Click(object sender, RoutedEventArgs e)
    => await _viewModel.SaveAsync();

public async Task SaveAsync()
    => await _repository.PersistAsync(_state);

public async Task PersistAsync(State s)
    => await _db.SaveChangesAsync().ConfigureAwait(false);
```

If you follow "async all the way," the deadlock **cannot happen** — there is no synchronous block anywhere on the path. `ConfigureAwait(false)` then becomes an optimization/safety net for libraries rather than a deadlock cure.

---

## Cancellation

Cooperative cancellation flows through `CancellationToken`. Accept one in every public async API and pass it down the chain.

```csharp
public async Task<string> DownloadAsync(string url, CancellationToken ct = default)
{
    using var response = await _http.GetAsync(url, ct).ConfigureAwait(false);
    response.EnsureSuccessStatusCode();
    return await response.Content.ReadAsStringAsync(ct).ConfigureAwait(false);
}
```

- Cancellation is **cooperative** — nothing is force-killed; the callee must observe the token.
- Combine tokens with `CancellationTokenSource.CreateLinkedTokenSource`.
- Use `CancellationTokenSource(TimeSpan)` for timeouts.
- A cancelled operation throws `OperationCanceledException` (or `TaskCanceledException`), which is normal control flow — don't treat it as an error.

---

## Exceptions in Async Code

- An exception in an `async Task` method is captured and **re-thrown when the task is awaited**.
- Awaiting a faulted task throws the **first** exception directly (not wrapped in `AggregateException`). Blocking via `.Result`/`.Wait()` wraps it in `AggregateException` — another reason to prefer `await`.
- Exceptions from `async void` **cannot be caught** by the caller; they are raised on the `SynchronizationContext` and typically **crash the process**.

```csharp
try
{
    await _service.LoadAsync();
}
catch (HttpRequestException ex)
{
    // The real exception, unwrapped.
    _logger.LogError(ex, "Load failed");
}
```

- Use `Task.WhenAll` carefully: it throws only the *first* exception when awaited, but the returned task's `.Exception` holds *all* of them. Inspect all faults when it matters.

---

## `async void` — When and Why to Avoid It

`async void` is a landmine because:

- Its exceptions escape to the `SynchronizationContext` and usually crash the app.
- It is **not awaitable** — callers cannot know when it finished or whether it failed.
- It breaks composition, testing, and reliable error handling.

**The only legitimate use is a UI event handler** whose signature is fixed by the framework (`void Button_Click(object, RoutedEventArgs)`). Even then, keep the body tiny and delegate to an awaitable method:

```csharp
private async void Button_Click(object sender, RoutedEventArgs e)
{
    try { await DoWorkAsync(); }
    catch (Exception ex) { ShowError(ex); }   // must catch here — no one else can
}
```

Everywhere else, return `Task`.

---

## Concurrency: Running Work in Parallel

Awaiting sequentially runs operations one after another. To run independent operations concurrently, start them, then await together:

```csharp
// ❌ Sequential — total time = A + B + C
var a = await GetAAsync();
var b = await GetBAsync();
var c = await GetCAsync();

// ✅ Concurrent — total time ≈ max(A, B, C)
var taskA = GetAAsync();
var taskB = GetBAsync();
var taskC = GetCAsync();
await Task.WhenAll(taskA, taskB, taskC);
var (a, b, c) = (taskA.Result, taskB.Result, taskC.Result); // safe: already completed
```

- `Task.WhenAll` — wait for all; aggregates results and exceptions.
- `Task.WhenAny` — wait for the first to finish (e.g. timeout races).
- For CPU-bound work, offload with `Task.Run`, but **do not** wrap I/O in `Task.Run` just to "make it async" — that wastes a thread.
- Bound concurrency (e.g. `SemaphoreSlim`) when fanning out many calls to avoid overwhelming a resource.

---

## Expert Pitfalls

- **`Task.Run` for I/O.** Offloading naturally-async I/O to a thread-pool thread reintroduces the very blocking you're trying to avoid. Only use `Task.Run` for CPU-bound work.
- **Fire-and-forget without observation.** Discarding a task (`_ = DoAsync();`) swallows exceptions and loses completion. If you must, wrap in a helper that logs faults.
- **`await` inside a `lock`.** Illegal and meaningless — you cannot hold a monitor across a suspension. Use `SemaphoreSlim.WaitAsync()` for async mutual exclusion.
- **Capturing `HttpContext`/request state after `ConfigureAwait(false)`.** You may be on a different thread; ambient state may be gone.
- **Elided async (returning the task) vs. awaiting.** Returning the task directly avoids state-machine overhead but changes exception timing and `using`/`try` scopes. Don't return a task from inside a `using` block — the resource is disposed before the task completes. Await it instead.
- **`ValueTask` misuse.** Awaiting twice or blocking on it is undefined behavior.
- **Long synchronous prefixes.** Everything *before* the first `await` runs synchronously on the caller's thread — heavy work there still blocks the UI.
- **Async lazy initialization.** Prefer `Lazy<Task<T>>` or `AsyncLazy` patterns over ad-hoc double-checked locking with async.
- **Deadlocks from constructors/property getters.** These can't be `async`, tempting people into `.Result`. Refactor to an async factory method instead.

---

## A Realistic Migration Strategy

When an existing **synchronous** application starts consuming a modernized, **async-based** library:

1. **Make the library correct first.** Apply `ConfigureAwait(false)` on **every** `await` throughout the library. This makes it deadlock-safe *even for callers that still block synchronously*.
2. **Let the app stay synchronous for now.** With the library fixed, the app may keep calling `.GetAwaiter().GetResult()` / `.Result` **without freezing**, as an interim step.
3. **Migrate the app incrementally.** Replace blocking calls with real `await`, propagating `async` upward one layer at a time until the whole path is "async all the way."

This lets you modernize without a risky big-bang rewrite, while never shipping a frozen UI in between.

---

## Guarding Against Regressions

Because a single forgotten `ConfigureAwait(false)` can bring the deadlock back, enforce the convention mechanically:

- **Document the rule** in your `CONTRIBUTING.md`, code-review checklist, and editor/Copilot instructions.
- **Enable the analyzer.** Rule **CA2007** ("Do not directly await a Task") flags every `await` that lacks a `ConfigureAwait`, so missing calls fail the build instead of shipping.

```xml
<!-- .editorconfig -->
<!-- Enforce ConfigureAwait in library projects -->
dotnet_diagnostic.CA2007.severity = warning
```

```xml
<!-- Or treat as error in the .csproj of a library -->
<PropertyGroup>
  <AnalysisMode>AllEnabledByDefault</AnalysisMode>
  <WarningsAsErrors>CA2007</WarningsAsErrors>
</PropertyGroup>
```

> Tip: Enable CA2007 in **library** projects, but consider disabling it in **UI/application** projects where *not* capturing the context would be wrong.

---

## Diagnosing a Frozen App

Signs that you're looking at a sync-over-async deadlock rather than a genuine hang:

- The **UI freezes completely** — menus and buttons stop responding — but the process has **not** crashed.
- Logs show the underlying (e.g. network) operation **succeeded**, yet the app never continues past it.
- The trigger is a `.Result`, `.Wait()`, or `.GetAwaiter().GetResult()` invoked **on the UI thread** (or a classic ASP.NET request thread).
- Pausing in the debugger shows the UI thread blocked inside a `WaitOne`/`Wait` call while a continuation sits pending on the context.

The fix is always one of: add `ConfigureAwait(false)` throughout the library, or (better) go async all the way and remove the blocking call.

---

## Checklist

- [ ] Public async methods return `Task`/`Task<T>`/`ValueTask`, never `void` (except UI event handlers).
- [ ] `async void` only on framework event handlers, with a `try/catch` inside.
- [ ] Library code uses `ConfigureAwait(false)` on **every** `await`.
- [ ] UI code does **not** use `ConfigureAwait(false)` when it must resume on the UI thread.
- [ ] No `.Result` / `.Wait()` / `.GetAwaiter().GetResult()` on a thread with a `SynchronizationContext`.
- [ ] `CancellationToken` accepted and forwarded through the chain.
- [ ] Independent operations run concurrently via `Task.WhenAll`, not awaited one by one.
- [ ] No `await` inside `lock`; use `SemaphoreSlim.WaitAsync`.
- [ ] No `Task.Run` wrapping naturally-async I/O.
- [ ] CA2007 enabled in library projects to enforce the convention.

---

## Further Reading

- [Asynchronous programming with async and await (Microsoft Docs)](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)
- [Async in depth (Microsoft Docs)](https://learn.microsoft.com/en-us/dotnet/standard/async-in-depth)
- [ConfigureAwait FAQ (Stephen Toub)](https://devblogs.microsoft.com/dotnet/configureawait-faq/)
- [CA2007: Do not directly await a Task](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca2007)
