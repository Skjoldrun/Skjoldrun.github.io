---
layout: page
title: C# - Parallelism vs Asynchrony & Process vs Thread
parent: C#
---

# Parallelism vs Asynchrony & Process vs Thread

These four terms are constantly mixed up, yet they describe two *different axes* of the same problem: **how work is executed** and **where it runs**. This guide separates them cleanly, shows how they relate, and gives concrete C# examples plus ASCII diagrams for the mental model.

- **Asynchrony vs Parallelism** answers: *"Do I wait, or do I do something else while waiting?"* and *"Does more than one thing happen at the exact same instant?"*
- **Process vs Thread** answers: *"What is the operating-system container my code runs in, and what memory does it share?"*

---

## Table of Contents

1. [The Big Picture](#the-big-picture)
2. [Process vs Thread](#process-vs-thread)
3. [Asynchrony vs Parallelism](#asynchrony-vs-parallelism)
4. [Concurrency: The Umbrella Term](#concurrency-the-umbrella-term)
5. [How They Combine](#how-they-combine)
6. [CPU-bound vs I/O-bound: The Decisive Question](#cpu-bound-vs-io-bound-the-decisive-question)
7. [Code Examples in C#](#code-examples-in-c)
8. [Common Misconceptions](#common-misconceptions)
9. [Decision Cheat Sheet](#decision-cheat-sheet)
10. [Summary](#summary)

---

## The Big Picture

Think of two independent questions:

```
                     WHERE does it run?
                 (Process / Thread axis)
                          ^
                          |
      Process A           |           Process B
   (isolated memory)      |        (isolated memory)
                          |
   Thread  Thread  Thread | Thread  Thread
                          |
--------------------------+--------------------------> HOW is it scheduled?
                          |          (Async / Parallel axis)
                          |
   Asynchronous: "start work, don't block, resume later"
   Parallel:     "run multiple things at the SAME instant"
```

- **Process / Thread** = the *containers* provided by the operating system.
- **Async / Parallel** = *strategies* for using those containers efficiently.

You can be asynchronous on a single thread. You can be parallel across many threads. You can run parallel work across many processes. They are orthogonal.

---

## Process vs Thread

### Definitions

| Aspect | Process | Thread |
|--------|---------|--------|
| Memory | Own isolated address space | Shares the process's memory |
| Creation cost | Heavy (MBs, OS bookkeeping) | Light (KB stack) |
| Communication | IPC: pipes, sockets, shared memory, files | Shared variables (fast, but needs locking) |
| Crash impact | Isolated — one crash doesn't kill others | A crash/corruption can take down the whole process |
| Owned by | The OS | A process (a process has 1..N threads) |
| Security boundary | Strong (separate address space) | None between threads of same process |

### ASCII Model

```
+-------------------------------------------------------+
|                     PROCESS (chrome.exe)              |
|                                                       |
|   Shared: heap, globals, file handles, loaded DLLs    |
|                                                       |
|   +-----------+   +-----------+   +-----------+       |
|   | Thread 1  |   | Thread 2  |   | Thread 3  |       |
|   |  stack    |   |  stack    |   |  stack    |       |
|   |  registers|   |  registers|   |  registers|       |
|   +-----------+   +-----------+   +-----------+       |
+-------------------------------------------------------+

+-------------------------------------------------------+
|                 SEPARATE PROCESS (notepad.exe)        |
|   Completely isolated memory — cannot touch chrome's  |
|   heap directly. Must use IPC to communicate.         |
+-------------------------------------------------------+
```

**Key insight:** Threads within one process share memory (fast communication, but you must synchronize with locks). Processes are isolated (safe, but communication is more expensive).

### In C#

```csharp
using System.Diagnostics;
using System.Threading;

// A THREAD (inside the current process — shares memory)
var worker = new Thread(() =>
{
    Console.WriteLine($"Worker thread id: {Environment.CurrentManagedThreadId}");
});
worker.Start();
worker.Join();

// A separate PROCESS (isolated — its own memory)
Process.Start(new ProcessStartInfo
{
    FileName = "dotnet",
    Arguments = "--version"
});
```

> In practice you rarely create raw `Thread` objects in modern C#. You use the **thread pool** via `Task.Run`, `Parallel`, or PLINQ, and let the runtime manage threads for you.

---

## Asynchrony vs Parallelism

This is the axis people confuse the most. The difference is **time**, not thread count.

### Asynchrony — "don't block while waiting"

Asynchrony is about **not blocking** a thread while some operation (often I/O) completes elsewhere. It does **not** imply multiple things happen at once — a single thread can juggle many async operations by starting them and resuming when results arrive.

```
Single thread, asynchronous (I/O-bound):

Thread: [start DB query]---(await: thread is FREE, does other work)---[resume with result]
                          \
                           `--> the DB does the actual waiting/work,
                                not our thread.

Time -------------------------------------------------->
```

The classic analogy: a **waiter** takes an order, sends it to the kitchen, and instead of standing there waiting, serves other tables. One waiter (one thread) handles many tables (many operations) because the *cooking* (I/O) happens elsewhere.

### Parallelism — "do many things at the same instant"

Parallelism is about **simultaneous execution** — literally multiple CPU cores doing work in the same instant. It requires multiple hardware execution units (cores).

```
Multiple threads, parallel (CPU-bound), on a 4-core CPU:

Core 1: [====== compute chunk A ======]
Core 2: [====== compute chunk B ======]
Core 3: [====== compute chunk C ======]
Core 4: [====== compute chunk D ======]
        ^ all four running at the SAME real instant

Time -------------------------------------------------->
```

The analogy: **four cooks** in the kitchen each preparing a different dish simultaneously. More cooks = more dishes finished per unit of time (assuming you have the burners/cores).

### Side-by-side

| | Asynchrony | Parallelism |
|--|-----------|-------------|
| Core idea | Don't block; resume later | Run simultaneously |
| Needs multiple cores? | No | Yes (for true simultaneity) |
| Best for | I/O-bound work (network, disk, DB) | CPU-bound work (math, image processing) |
| Threads used | Can be a single thread | Multiple threads |
| C# tools | `async`/`await`, `Task` | `Parallel`, PLINQ, `Task.Run` fan-out |
| Goal | Scalability / responsiveness | Throughput / speed |

---

## Concurrency: The Umbrella Term

**Concurrency** = *dealing with* many things at once (structure). **Parallelism** = *doing* many things at once (execution).

```
Concurrency (structure): tasks make progress in overlapping time windows.

Task A: [--]      [--]        [----]
Task B:     [----]    [--]
Task C:                   [--]      [--]
        (interleaved on possibly ONE core — no true simultaneity required)

Parallelism (execution): tasks run at the same physical instant.

Task A: [==========]   (core 1)
Task B: [==========]   (core 2)
        ^ same instant
```

- Concurrency **can** exist without parallelism (async single thread, or time-sliced threads on one core).
- Parallelism is a **specific form** of concurrency backed by multiple cores.

---

## How They Combine

All four concepts stack on top of each other:

```
Process
  └── Thread(s)
        └── run work either:
              - Asynchronously (start, await, resume — great for I/O)
              - In Parallel (many threads/cores at once — great for CPU)
```

Real examples:

- **Async on one thread:** A web server handling 10,000 simultaneous connections with `async`/`await` — most are just waiting on the network, so a handful of threads suffice.
- **Parallel across threads:** Resizing 1,000 images using `Parallel.ForEach` to saturate all CPU cores.
- **Parallel across processes:** A CI system running test suites in separate worker processes for isolation.
- **Async + Parallel together:** Fetch 100 URLs concurrently (async I/O), then CPU-parallel-process each downloaded payload.

---

## CPU-bound vs I/O-bound: The Decisive Question

This single distinction usually tells you which tool to reach for.

```
Is the work waiting on something external (network/disk/DB)?
        |
        +-- YES (I/O-bound)  --> use ASYNC (async/await). Threads stay free.
        |
        +-- NO  (CPU-bound)  --> use PARALLELISM (Parallel/PLINQ/Task.Run).
                                 Spread the compute across cores.
```

**Anti-pattern:** Using `Task.Run` to wrap I/O just to "make it async" wastes a thread-pool thread that then blocks on I/O — the opposite of what you want. Use real async I/O APIs (`HttpClient.GetAsync`, `stream.ReadAsync`, etc.) instead.

---

## Code Examples in C#

### 1. Asynchronous (I/O-bound) — one thread stays free

```csharp
using System.Net.Http;

async Task<string> DownloadAsync(HttpClient http, string url)
{
    // While the server responds, THIS thread is returned to the pool
    // and can serve other requests. No thread is blocked "waiting".
    HttpResponseMessage resp = await http.GetAsync(url);
    return await resp.Content.ReadAsStringAsync();
}

// Fire off many downloads concurrently — still I/O-bound, few threads used.
async Task DownloadManyAsync()
{
    using var http = new HttpClient();
    string[] urls = { "https://example.com", "https://example.org" };

    // Start all, then await all — they overlap in time (concurrency)
    Task<string>[] downloads = urls.Select(u => DownloadAsync(http, u)).ToArray();
    string[] pages = await Task.WhenAll(downloads);

    Console.WriteLine($"Downloaded {pages.Length} pages.");
}
```

### 2. Parallel (CPU-bound) — many cores at once

```csharp
using System.Threading.Tasks;

// Heavy pure-CPU work spread across all cores.
long[] numbers = Enumerable.Range(1, 1_000_000).Select(i => (long)i).ToArray();

// Parallel.ForEach schedules chunks onto multiple thread-pool threads,
// which the OS maps onto multiple CPU cores => true parallelism.
long total = 0;
Parallel.ForEach(
    numbers,
    () => 0L,                          // thread-local seed
    (n, _, localSum) => localSum + IsPrimeCost(n), // per-element work
    localSum => Interlocked.Add(ref total, localSum)); // combine safely

static long IsPrimeCost(long n) => n % 2; // stand-in for expensive compute
```

### 3. PLINQ — declarative parallelism

```csharp
using System.Linq;

var results = numbers
    .AsParallel()                 // opt into parallel execution
    .Where(n => n % 7 == 0)
    .Select(n => n * n)
    .ToArray();
```

### 4. Async is NOT parallelism — proof on one thread

```csharp
// These two awaits run on (potentially) the same single thread.
// They are concurrent (overlapping waits) but not parallel:
// no two lines of YOUR code execute at the same instant.
async Task NotParallelAsync()
{
    Task a = Task.Delay(1000); // timer runs in the OS, not on a thread
    Task b = Task.Delay(1000);
    await Task.WhenAll(a, b);   // both finish in ~1s total, one thread suffices
    Console.WriteLine("Both delays done — no CPU threads were burned waiting.");
}
```

### 5. Process — isolation for safety

```csharp
using System.Diagnostics;

// Run risky/heavy work in a SEPARATE process so a crash can't
// corrupt or take down the parent's memory.
var psi = new ProcessStartInfo("dotnet", "run --project ./Worker")
{
    RedirectStandardOutput = true,
    UseShellExecute = false
};
using var proc = Process.Start(psi)!;
string output = proc.StandardOutput.ReadToEnd();
proc.WaitForExit();
```

---

## Common Misconceptions

- **"Async means multithreaded."** No. `await` can complete on the same thread; for pure I/O, no dedicated thread waits at all.
- **"More threads = faster."** Only for CPU-bound work up to the core count. For I/O-bound work, more threads just add context-switching overhead — async scales far better.
- **"Parallelism makes any code faster."** Only CPU-bound, divisible work benefits. Parallelizing I/O or tiny tasks can be *slower* due to overhead and contention.
- **"Threads are cheap, spin up thousands."** Each thread has a ~1MB stack and scheduling cost. Thousands of blocked threads is a classic scalability killer; async avoids it.
- **"Processes and threads are basically the same."** They differ fundamentally in memory isolation and crash blast-radius.

---

## Decision Cheat Sheet

| I want to… | Reach for… |
|------------|-----------|
| Keep a UI responsive while loading data | `async`/`await` |
| Handle thousands of network connections | `async`/`await` (I/O-bound) |
| Crunch a big array / image processing | `Parallel` / PLINQ (CPU-bound) |
| Isolate untrusted or crash-prone work | Separate **process** |
| Share large in-memory state cheaply | **Threads** (with locks) in one process |
| Overlap many independent waits | Concurrency via `Task.WhenAll` |

---

## Summary

```
                    +-----------------------------+
                    |        CONCURRENCY          |
                    |  (dealing with many things) |
                    +--------------+--------------+
                                   |
              +--------------------+--------------------+
              |                                         |
      ASYNCHRONY                                  PARALLELISM
  "don't block while waiting"              "run at the same instant"
   best for I/O-bound                       best for CPU-bound
   can use ONE thread                       needs MANY cores/threads
              |                                         |
              +--------------------+--------------------+
                                   |
                          runs inside THREADS
                                   |
                          which live in a PROCESS
                          (isolated memory container)
```

- **Process vs Thread** = *where* code runs and *what memory* it shares (isolation vs sharing).
- **Asynchrony vs Parallelism** = *how* work is scheduled (don't-block vs same-instant).
- Match the tool to the workload: **async for I/O-bound, parallelism for CPU-bound**, and reach for **separate processes when you need isolation**.
