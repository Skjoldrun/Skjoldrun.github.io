---
layout: page
title: C# - XML Doc Comments, Summary & Inheritdoc
parent: C#
---

# XML Documentation Comments in C#

C# lets you document your code directly in the source using **XML documentation
comments**. They start with a triple slash (`///`) and sit right above the type
or member they describe. The compiler can extract them into an XML file, IDEs
use them for IntelliSense tooltips, and tools like DocFX or Sandcastle turn them
into full API documentation websites.

This article focuses on `<summary>` comments and the very handy `<inheritdoc>`
tag, plus a couple of related tags that are worth knowing.

## Summary Comments

The `<summary>` tag is the most common and most important documentation tag. It
gives a short description of what a type or member does. This is the text that
shows up in IntelliSense when you hover over a symbol.

```csharp
/// <summary>
/// Represents a bank account and provides operations to deposit and withdraw money.
/// </summary>
public class BankAccount
{
    /// <summary>
    /// Gets the current balance of the account.
    /// </summary>
    public decimal Balance { get; private set; }

    /// <summary>
    /// Deposits the specified amount into the account.
    /// </summary>
    /// <param name="amount">The amount to deposit. Must be greater than zero.</param>
    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentOutOfRangeException(nameof(amount));

        Balance += amount;
    }
}
```

Keep summaries short and to the point. Describe *what* something does, not *how*
it is implemented. If a member needs a longer explanation, use `<remarks>` for
the additional detail.

## Common Companion Tags

A `<summary>` rarely stands alone. These tags round out the documentation:

- `<param name="...">` – describes a single method parameter.
- `<returns>` – describes the return value of a method.
- `<typeparam name="...">` – describes a generic type parameter.
- `<exception cref="...">` – documents an exception a member may throw.
- `<remarks>` – additional information that does not fit in the summary.
- `<example>` – a usage example, often combined with `<code>`.
- `<see cref="..."/>` and `<seealso cref="..."/>` – cross references to other members.
- `<paramref name="..."/>` – references a parameter by name inside prose.

```csharp
/// <summary>
/// Divides one number by another.
/// </summary>
/// <param name="dividend">The number to be divided.</param>
/// <param name="divisor">The number to divide by.</param>
/// <returns>The result of dividing <paramref name="dividend"/> by <paramref name="divisor"/>.</returns>
/// <exception cref="DivideByZeroException">
/// Thrown when <paramref name="divisor"/> is zero.
/// </exception>
/// <example>
/// <code>
/// double result = Calculator.Divide(10, 2); // 5
/// </code>
/// </example>
public static double Divide(double dividend, double divisor)
{
    if (divisor == 0)
        throw new DivideByZeroException();

    return dividend / divisor;
}
```

## Inheritdoc

When a class implements an interface or overrides a base member, you often want
the documentation to be exactly the same. Copy-pasting the comments works, but
then you have to keep every copy in sync. That is what **`<inheritdoc/>`** solves:
it tells tools to reuse the documentation from the base type, interface, or
overridden member.

### Inheriting from an interface

```csharp
/// <summary>
/// Provides read access to a data store.
/// </summary>
public interface IRepository<T>
{
    /// <summary>
    /// Gets the entity with the specified identifier.
    /// </summary>
    /// <param name="id">The unique identifier of the entity.</param>
    /// <returns>The matching entity, or <c>null</c> if none exists.</returns>
    T? GetById(int id);
}

/// <inheritdoc />
public class UserRepository : IRepository<User>
{
    /// <inheritdoc />
    public User? GetById(int id)
    {
        // Implementation...
        return null;
    }
}
```

Both the class summary and the `GetById` documentation are pulled from
`IRepository<T>`. No duplication, and the docs never drift apart.

### Inheriting when overriding a base member

```csharp
public class Animal
{
    /// <summary>
    /// Produces the sound this animal makes.
    /// </summary>
    public virtual string Speak() => "...";
}

public class Dog : Animal
{
    /// <inheritdoc />
    public override string Speak() => "Woof";
}
```

### Selecting a specific source with `cref`

When a member could inherit from several places (for example, a class that
implements multiple interfaces), you can point `<inheritdoc>` at a specific
source with the `cref` attribute:

```csharp
/// <inheritdoc cref="IRepository{T}.GetById(int)" />
public User? GetById(int id) => null;
```

### Inheriting only part of the documentation with `path`

`<inheritdoc>` supports an XPath expression through the `path` attribute, so you
can inherit selected tags and still override others:

```csharp
/// <inheritdoc cref="Animal.Speak" path="/summary" />
/// <returns>Always returns "Woof".</returns>
public override string Speak() => "Woof";
```

Here the `<summary>` is inherited from `Animal.Speak`, while the `<returns>` is
written locally.

## Enabling and Enforcing Documentation

To generate the XML documentation file during a build, enable it in the project
file. You can also turn missing-comment warnings on or off:

```xml
<PropertyGroup>
  <GenerateDocumentationFile>true</GenerateDocumentationFile>

  <!-- 1591: "Missing XML comment for publicly visible type or member" -->
  <NoWarn>$(NoWarn);1591</NoWarn>
</PropertyGroup>
```

- Set `GenerateDocumentationFile` to `true` to emit `YourAssembly.xml` next to
  the compiled DLL. This file ships alongside the assembly so consumers get
  IntelliSense.
- Warning **CS1591** flags public members without documentation. Suppress it via
  `NoWarn` while you are still filling in comments, or treat it as an error to
  enforce complete docs on public APIs.

> **Note:** `<inheritdoc>` is understood by the Roslyn compiler, modern IDEs
> (Visual Studio, Rider) and documentation generators such as DocFX and
> Sandcastle. Very old toolchains may not resolve it, so check your doc pipeline
> if the inherited text does not appear.

## Best Practices

- Document all **public** and **protected** members – they form your API surface.
- Keep `<summary>` concise; move detail into `<remarks>`.
- Start method summaries with a verb: *"Gets…"*, *"Creates…"*, *"Sends…"*.
- Use `<inheritdoc/>` for overrides and interface implementations to avoid
  duplicated, drifting comments.
- Use `<see cref="..."/>` for links – you get compile-time checked references
  that survive renames.
- Write comments in English so they are useful to the widest audience.

## Summary

XML documentation comments make your code self-describing. `<summary>` provides
the core description shown in IntelliSense, the companion tags (`<param>`,
`<returns>`, `<exception>`, …) fill in the details, and `<inheritdoc>` keeps
inherited and overridden members documented without duplication. Enable
`GenerateDocumentationFile` to ship this information with your assemblies and
give consumers a first-class development experience.
