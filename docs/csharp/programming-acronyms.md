---
layout: page
title: C# - Programming Acronyms & Abbreviations
parent: C#
---

# Programming Acronyms & Abbreviations — A Glossary with C# Examples

> The software world loves abbreviations. **DRY**, **YAGNI**, **KISS** and friends are more than cute letters — they are compressed wisdom that helps you write better code. This article explains the most important acronyms, spells them out, describes what they mean, and — where useful — shows concrete examples in C#.

---

## Table of Contents

1. [Design & Coding Principles](#design--coding-principles)
   - [DRY — Don't Repeat Yourself](#dry--dont-repeat-yourself)
   - [WET — Write Everything Twice](#wet--write-everything-twice)
   - [KISS — Keep It Simple, Stupid](#kiss--keep-it-simple-stupid)
   - [YAGNI — You Aren't Gonna Need It](#yagni--you-arent-gonna-need-it)
   - [SoC — Separation of Concerns](#soc--separation-of-concerns)
   - [SOLID](#solid)
   - [POLA / POLS — Principle of Least Astonishment/Surprise](#pola--pols--principle-of-least-astonishmentsurprise)
   - [LoD — Law of Demeter](#lod--law-of-demeter)
   - [CoC — Convention over Configuration](#coc--convention-over-configuration)
2. [Architecture & Patterns](#architecture--patterns)
   - [MVC / MVVM / MVP](#mvc--mvvm--mvp)
   - [DI / IoC — Dependency Injection / Inversion of Control](#di--ioc--dependency-injection--inversion-of-control)
   - [CQRS — Command Query Responsibility Segregation](#cqrs--command-query-responsibility-segregation)
   - [DDD — Domain-Driven Design](#ddd--domain-driven-design)
   - [DTO / POCO / VO](#dto--poco--vo)
   - [ORM — Object-Relational Mapping](#orm--object-relational-mapping)
3. [Quality & Process](#quality--process)
   - [TDD / BDD — Test-Driven / Behavior-Driven Development](#tdd--bdd--test-driven--behavior-driven-development)
   - [AAA — Arrange, Act, Assert](#aaa--arrange-act-assert)
   - [CI / CD — Continuous Integration / Continuous Delivery](#ci--cd--continuous-integration--continuous-delivery)
   - [MVP — Minimum Viable Product](#mvp--minimum-viable-product)
   - [PoC — Proof of Concept](#poc--proof-of-concept)
   - [WIP — Work In Progress](#wip--work-in-progress)
   - [RTFM — Read The Friendly Manual](#rtfm--read-the-friendly-manual)
4. [Technical Design & Data World](#technical-design--data-world)
   - [CRUD — Create, Read, Update, Delete](#crud--create-read-update-delete)
   - [ACID vs. BASE](#acid-vs-base)
   - [REST / SOAP / RPC](#rest--soap--rpc)
   - [JSON / XML / YAML](#json--xml--yaml)
   - [FIFO / LIFO](#fifo--lifo)
   - [Big-O / O(n)](#big-o--on)
5. [Slang & Warning Signs](#slang--warning-signs)
   - [Code Smell, Tech Debt, WTF/min](#code-smell-tech-debt-wtfmin)
   - [FUBAR / SNAFU](#fubar--snafu)
   - [PEBKAC / ID10T](#pebkac--id10t)
   - [Foo / Bar / Baz](#foo--bar--baz)
6. [Cheat Sheet](#cheat-sheet)

---

## Design & Coding Principles

### DRY — Don't Repeat Yourself

> Every piece of knowledge (logic, configuration, rule) should have exactly **one** unambiguous representation in the system.

When the same logic is copied to multiple places, a change forces you to touch *all* copies — and you will surely forget one. DRY says: extract what's common into a single place.

```csharp
// ❌ Not DRY — the same discount logic is duplicated
public decimal CalculateInvoiceTotal(decimal amount)
{
    return amount - (amount * 0.1m); // 10% discount
}

public decimal CalculateQuoteTotal(decimal amount)
{
    return amount - (amount * 0.1m); // the same 10% — what if the value changes?
}
```

```csharp
// ✅ DRY — a single source of truth
public static class Discount
{
    public const decimal StandardRate = 0.1m;

    public static decimal Apply(decimal amount) => amount - (amount * StandardRate);
}

public decimal CalculateInvoiceTotal(decimal amount) => Discount.Apply(amount);
public decimal CalculateQuoteTotal(decimal amount) => Discount.Apply(amount);
```

> ⚠️ **Beware of over-applying DRY:** Two pieces of code that *happen* to look alike but follow different business reasons should stay separate. Otherwise you couple things that are actually independent.

---

### WET — Write Everything Twice

> The ironic counterpart to DRY (also read as *"We Enjoy Typing"* or *"Waste Everyone's Time"*). It describes code that is needlessly duplicated.

WET is mostly a warning label: saying "this code is WET" means it violates DRY. The related *Rule of Three* is often cited: only when you write the same thing for the **third** time is the abstraction worth it — before that you risk a wrong abstraction.

---

### KISS — Keep It Simple, Stupid

> The simplest solution that solves the problem is usually the best one. Complexity is a cost, not a quality.

```csharp
// ❌ Needlessly complex
public bool IsEven(int number)
{
    return Convert.ToBoolean(Math.Abs(number % 2 - 1));
}

// ✅ KISS
public bool IsEven(int number) => number % 2 == 0;
```

---

### YAGNI — You Aren't Gonna Need It

> Don't build features or flexibility "just in case" simply because you *might* need them someday.

YAGNI fights over-engineering. Every line of code you write today for a hypothetical tomorrow must be maintained, tested, and understood — usually for nothing.

```csharp
// ❌ YAGNI violation — a provider framework for ONE use case
public interface IPaymentProvider { void Pay(decimal amount); }
public class PayPalProvider : IPaymentProvider { /* ... */ }
public class BitcoinProvider : IPaymentProvider { /* ... */ }
public class GoldBarProvider : IPaymentProvider { /* ... */ } // nobody asked for this!

// ✅ YAGNI — solve the real problem, extend only when needed
public class PaymentService
{
    public void Charge(decimal amount) => Console.WriteLine($"Charged {amount:C}");
}
```

> DRY, KISS, and YAGNI together form the "holy trinity" of pragmatic software development.

---

### SoC — Separation of Concerns

> Different concerns (UI, business logic, data access) belong in different, clearly separated areas.

SoC is the overarching principle behind layered architectures, MVC, and the Single Responsibility Principle. Don't mix database queries with HTML output.

---

### SOLID

> Five object-oriented design principles by Robert C. Martin (Uncle Bob).

| Letter | Spelled Out | Meaning |
|--------|-------------|---------|
| **S** | Single Responsibility Principle | One class, one reason to change |
| **O** | Open/Closed Principle | Open for extension, closed for modification |
| **L** | Liskov Substitution Principle | Subclasses must honor the base class contract |
| **I** | Interface Segregation Principle | Small, focused interfaces instead of one "fat" interface |
| **D** | Dependency Inversion Principle | Depend on abstractions, not implementations |

> 📖 Explained in depth with Star Wars examples in the dedicated article [SOLID principles](solid.md).

---

### POLA / POLS — Principle of Least Astonishment/Surprise

> A component should behave the way most users would expect. No hidden side effects.

```csharp
// ❌ Surprising — a getter mutates state!
public int Count
{
    get { _accessCount++; return _items.Count; } // who expects that?
}

// ✅ Expected — a getter only reads
public int Count => _items.Count;
```

---

### LoD — Law of Demeter

> "Only talk to your friends." An object should communicate only with its direct neighbors, not reach through long chains of foreign objects.

```csharp
// ❌ Violation — reaching through several layers ("train wreck")
var zip = order.Customer.Address.PostalCode.Value;

// ✅ The object exposes what you need
var zip = order.GetShippingPostalCode();
```

---

### CoC — Convention over Configuration

> Sensible default assumptions reduce configuration effort. Only deviations from the default need to be configured.

Example: In ASP.NET Core the router maps `HomeController.Index()` to `/Home/Index` by default — without you defining every route explicitly.

---

## Architecture & Patterns

### MVC / MVVM / MVP

> Three related presentation patterns that separate UI from logic.

| Acronym | Spelled Out | Typical Use |
|---------|-------------|-------------|
| **MVC** | Model-View-Controller | ASP.NET Core web apps |
| **MVVM** | Model-View-ViewModel | WPF, MAUI, Blazor |
| **MVP** | Model-View-Presenter | WinForms, older UI frameworks |

Shared idea: the **Model** holds the data, the **View** displays it, and a middle layer (Controller / ViewModel / Presenter) mediates between them.

---

### DI / IoC — Dependency Injection / Inversion of Control

> A class does not create its dependencies itself but receives them from the outside. This makes code testable and loosely coupled.

```csharp
// ❌ Without DI — the class creates its dependency itself (tightly coupled)
public class ReportService
{
    private readonly SqlLogger _logger = new SqlLogger(); // not swappable
}

// ✅ With DI — the dependency is injected through the constructor
public class ReportService
{
    private readonly ILogger _logger;
    public ReportService(ILogger logger) => _logger = logger; // testable, swappable
}

// Registration in the DI container (e.g. ASP.NET Core)
services.AddScoped<ILogger, SqlLogger>();
services.AddScoped<ReportService>();
```

> **IoC** is the overarching principle ("don't call the framework, the framework calls you"); **DI** is its concrete implementation.

---

### CQRS — Command Query Responsibility Segregation

> Write operations (commands) and read operations (queries) are handled through separate models.

```csharp
// Command — changes state, ideally returns nothing
public record CreateOrderCommand(int CustomerId, string Product);

// Query — reads state, changes nothing
public record GetOrderByIdQuery(int OrderId);
```

Useful in complex domains, often combined with Event Sourcing and libraries such as MediatR.

---

### DDD — Domain-Driven Design

> The code models the business domain using a shared vocabulary (*Ubiquitous Language*), with Entities, Value Objects, Aggregates, and Bounded Contexts.

Goal: the structure of the software reflects the real business problem — not the technical database.

---

### DTO / POCO / VO

> Three often-confused object types.

| Acronym | Spelled Out | Purpose |
|---------|-------------|---------|
| **DTO** | Data Transfer Object | Transports data between layers/services, no logic |
| **POCO** | Plain Old CLR Object | Simple class without framework dependencies |
| **VO** | Value Object | Defined by its values, immutable, no identity |

```csharp
// DTO — a pure data container for the API response
public record CustomerDto(int Id, string Name, string Email);

// Value Object — equality by value, not by reference
public record Money(decimal Amount, string Currency);

var a = new Money(10m, "EUR");
var b = new Money(10m, "EUR");
Console.WriteLine(a == b); // True — records compare by value
```

> **POCO** is the .NET equivalent of Java's **POJO** (Plain Old Java Object).

---

### ORM — Object-Relational Mapping

> An ORM translates between objects in code and tables in a relational database, so you rarely have to write raw SQL.

In the .NET world: **EF Core** (Entity Framework Core) and **Dapper** (a micro-ORM).

```csharp
// With EF Core: LINQ instead of SQL
var adults = await db.Users
    .Where(u => u.Age >= 18)
    .OrderBy(u => u.Name)
    .ToListAsync();
```

---

## Quality & Process

### TDD / BDD — Test-Driven / Behavior-Driven Development

> With **TDD** you write the test *before* the code (Red → Green → Refactor). **BDD** phrases tests in business language (Given/When/Then).

```csharp
// TDD cycle: 1. Write the test (fails = Red)
[Fact]
public void Add_TwoNumbers_ReturnsSum()
{
    var calc = new Calculator();
    Assert.Equal(5, calc.Add(2, 3)); // red first, then green
}
```

BDD style (Given/When/Then): *Given* an empty cart, *When* an item is added, *Then* the cart contains one item.

---

### AAA — Arrange, Act, Assert

> The standard layout for readable unit tests.

```csharp
[Fact]
public void Withdraw_ReducesBalance()
{
    // Arrange — prepare the object under test and data
    var account = new BankAccount(balance: 100);

    // Act — perform the action being tested
    account.Withdraw(30);

    // Assert — verify the result
    Assert.Equal(70, account.Balance);
}
```

---

### CI / CD — Continuous Integration / Continuous Delivery

> **CI** automatically integrates and tests code changes on every commit. **CD** automates deployment to test or production environments.

Common tools: GitHub Actions, Azure DevOps, GitLab CI. Depending on maturity, the second "D" stands for *Delivery* (deployable, but with a manual release button) or *Deployment* (fully automated to production).

---

### MVP — Minimum Viable Product

> The smallest version of a product that delivers real value and lets you learn from users.

> ⚠️ Watch out for the ambiguity: **MVP** means both *Minimum Viable Product* (a product strategy) and *Model-View-Presenter* (a UI pattern). Context tells you which is meant.

---

### PoC — Proof of Concept

> A small throwaway prototype that only proves an approach works technically — not production-ready.

---

### WIP — Work In Progress

> Marks unfinished work, e.g. in Git commits (`WIP: refactor auth`) or pull-request titles, to signal: not ready to merge yet.

---

### RTFM — Read The Friendly Manual

> Usually a slightly annoyed reply to questions whose answer is right there in the documentation. A reminder to check the docs first.

---

## Technical Design & Data World

### CRUD — Create, Read, Update, Delete

> The four basic operations on persistent data. Almost every data application is, at its core, a CRUD app.

| Operation | HTTP (REST) | SQL |
|-----------|-------------|-----|
| Create | POST | INSERT |
| Read | GET | SELECT |
| Update | PUT / PATCH | UPDATE |
| Delete | DELETE | DELETE |

```csharp
public interface IRepository<T>
{
    Task<T> CreateAsync(T entity);
    Task<T?> ReadAsync(int id);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
}
```

---

### ACID vs. BASE

> Two opposing consistency philosophies for databases.

**ACID** (classic relational DBs, transaction guarantees):

| Letter | Spelled Out | Meaning |
|--------|-------------|---------|
| **A** | Atomicity | All or nothing |
| **C** | Consistency | Valid state before and after the transaction |
| **I** | Isolation | Transactions don't interfere with each other |
| **D** | Durability | Committed data stays permanently |

**BASE** (many NoSQL systems, scalability over immediate consistency):

- **BA** — Basically Available
- **S** — Soft state
- **E** — Eventually consistent

---

### REST / SOAP / RPC

> Three styles for communication between distributed systems.

| Acronym | Spelled Out | Short Character |
|---------|-------------|-----------------|
| **REST** | Representational State Transfer | Resources over HTTP verbs, mostly JSON |
| **SOAP** | Simple Object Access Protocol | XML-based, strongly typed, with a schema (WSDL) |
| **RPC** | Remote Procedure Call | Calling remote methods (e.g. gRPC) |

---

### JSON / XML / YAML

> Three widespread data formats for serialization and configuration.

| Acronym | Spelled Out | Typical Use |
|---------|-------------|-------------|
| **JSON** | JavaScript Object Notation | APIs, configuration (`appsettings.json`) |
| **XML** | eXtensible Markup Language | Legacy, `.csproj`, SOAP |
| **YAML** | YAML Ain't Markup Language | CI pipelines, Docker Compose, Kubernetes |

> **YAML** is a so-called *recursive acronym* — its definition contains itself.

---

### FIFO / LIFO

> Two ordering strategies for data structures.

- **FIFO** — First In, First Out → **Queue**. Whoever comes first is served first.
- **LIFO** — Last In, First Out → **Stack**. The last item placed is the first one taken out.

```csharp
var queue = new Queue<string>();        // FIFO
queue.Enqueue("A"); queue.Enqueue("B");
Console.WriteLine(queue.Dequeue());     // "A"

var stack = new Stack<string>();        // LIFO
stack.Push("A"); stack.Push("B");
Console.WriteLine(stack.Pop());         // "B"
```

---

### Big-O / O(n)

> **Big-O notation** describes how the runtime or memory of an algorithm grows as the input size `n` increases.

| Notation | Name | Example |
|----------|------|---------|
| O(1) | constant | Index access in an array |
| O(log n) | logarithmic | Binary search |
| O(n) | linear | Loop over a list |
| O(n log n) | linearithmic | Efficient sorting algorithms |
| O(n²) | quadratic | Nested loops |

```csharp
// O(n) — touch each element exactly once
public bool Contains(int[] numbers, int target)
{
    foreach (var n in numbers)   // grows linearly with the length
        if (n == target) return true;
    return false;
}
```

---

## Slang & Warning Signs

### Code Smell, Tech Debt, WTF/min

- **Code Smell** — a "smell" in the code: not a bug, but a hint of deeper design problems (e.g. very long methods, duplication).
- **Tech Debt** (*Technical Debt*) — shortcuts in the design that save time short-term but must be paid back later "with interest".
- **WTF/min** — a half-ironic metric for code quality: the number of horrified exclamations per minute during a code review.

---

### FUBAR / SNAFU

> Terms for "broken" that come from military slang.

- **SNAFU** — *Situation Normal: All Fouled Up* → It's chaotic, but sadly that's the normal state.
- **FUBAR** — *Fouled Up Beyond All Recognition* → Completely destroyed, beyond saving. Often used for hopelessly messed-up code.

---

### PEBKAC / ID10T

> Joking diagnoses in IT support when the problem sits in front of the screen.

- **PEBKAC** — *Problem Exists Between Keyboard And Chair*.
- **ID10T** — disguised as an "error code"; read out loud it spells "IDIOT".
- Related: **Layer 8 problem** (the user as the eighth layer above the OSI model).

---

### Foo / Bar / Baz

> Classic **placeholder names** (*metasyntactic variables*) in example code when the actual name doesn't matter. `foo`, `bar`, and `baz` have no meaning — they simply stand for "anything".

```csharp
public void Example()
{
    var foo = GetSomething();
    var bar = Transform(foo);
    Console.WriteLine(bar);
}
```

> In example code you'll also often see names like `blah`, or `John Doe` for a placeholder person.

---

## Cheat Sheet

| Abbreviation | Spelled Out | Core Idea |
|--------------|-------------|-----------|
| **DRY** | Don't Repeat Yourself | Each piece of logic in only one place |
| **WET** | Write Everything Twice | Ironic opposite of DRY |
| **KISS** | Keep It Simple, Stupid | As simple as possible |
| **YAGNI** | You Aren't Gonna Need It | Don't build on spec |
| **SoC** | Separation of Concerns | Separate responsibilities |
| **SOLID** | 5 OO principles | Clean object-oriented design |
| **POLA** | Principle of Least Astonishment | No surprises |
| **LoD** | Law of Demeter | Only talk to direct neighbors |
| **CoC** | Convention over Configuration | Sensible defaults |
| **DI / IoC** | Dependency Injection / Inversion of Control | Dependencies from the outside |
| **CQRS** | Command Query Responsibility Segregation | Separate reads and writes |
| **DDD** | Domain-Driven Design | Code models the business domain |
| **DTO** | Data Transfer Object | Data transport without logic |
| **POCO** | Plain Old CLR Object | Simple class without framework baggage |
| **ORM** | Object-Relational Mapping | Objects ↔ database tables |
| **TDD / BDD** | Test-/Behavior-Driven Development | Tests first |
| **AAA** | Arrange, Act, Assert | Structure of unit tests |
| **CI / CD** | Continuous Integration / Delivery | Automated build & release |
| **MVP** | Minimum Viable Product | Smallest useful version |
| **PoC** | Proof of Concept | Feasibility check |
| **CRUD** | Create, Read, Update, Delete | Basic operations on data |
| **ACID** | Atomicity, Consistency, Isolation, Durability | Transaction guarantees |
| **BASE** | Basically Available, Soft state, Eventually consistent | NoSQL consistency |
| **REST** | Representational State Transfer | Resources over HTTP |
| **FIFO / LIFO** | First/Last In, First Out | Queue / Stack |
| **Big-O** | Big-O notation | Runtime/memory complexity |

> *Acronyms are tools, not dogmas.* Use them as a shared language in your team — but always understand the principle behind one before you apply it.
