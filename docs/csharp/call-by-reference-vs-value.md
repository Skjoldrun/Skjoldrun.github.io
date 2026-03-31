---
layout: page
title: C# - Call by Reference vs Call by Value
parent: C#
---

# Call by Reference vs Call by Value

In programming, when you pass arguments to a method, there are two fundamental ways the data can be handed over: **call by value** (a copy of the data is passed) and **call by reference** (a reference to the original data is passed). Understanding this distinction is essential for writing correct and predictable C# code, especially when dealing with mutations inside methods.


## Default Behavior in C#

C# uses **call by value** as the default for all parameter passing. But what "value" means depends on the type:

- **Value types** (`int`, `double`, `bool`, `struct`, `enum`, etc.): The actual data is copied. Changes inside the method do not affect the caller.
- **Reference types** (`class`, `string`, `arrays`, `interface`, `delegate`): The **reference** (pointer) is copied, not the object itself. This means the method can modify the object's properties (because both references point to the same object), but **reassigning** the reference inside the method does not affect the caller's variable.

This is a common source of confusion: reference types are still passed **by value** by default — it's just that the "value" being copied is the reference, not the object.


## Value Types — Pass by Value

```csharp
static void Increment(int number)
{
    number += 10;
    Console.WriteLine($"Inside method: {number}"); // 20
}

static void Main()
{
    int myNumber = 10;
    Increment(myNumber);
    Console.WriteLine($"After method call: {myNumber}"); // 10 — unchanged
}
```

The method receives a **copy** of `myNumber`. Modifying `number` inside the method has no effect on the caller's `myNumber`.


## Reference Types — Pass by Value (Default)

```csharp
class Person
{
    public string Name { get; set; }
}

static void ChangeName(Person person)
{
    person.Name = "Alice"; // modifies the original object
}

static void ReassignPerson(Person person)
{
    person = new Person { Name = "Bob" }; // only changes the local copy of the reference
}

static void Main()
{
    var p = new Person { Name = "Original" };

    ChangeName(p);
    Console.WriteLine(p.Name); // "Alice" — property was modified

    ReassignPerson(p);
    Console.WriteLine(p.Name); // "Alice" — reassignment did NOT affect the caller
}
```

`ChangeName` modifies the object's property through the copied reference — both the caller and the method point to the same object. `ReassignPerson` reassigns the local reference to a new object, but the caller's `p` still points to the original.


## The `ref` Keyword

The `ref` keyword passes a variable **by reference**. The method receives a direct alias to the caller's variable, so any modification — including reassignment — is visible to the caller.

```csharp
static void Increment(ref int number)
{
    number += 10;
}

static void ReassignPerson(ref Person person)
{
    person = new Person { Name = "Bob" };
}

static void Main()
{
    int myNumber = 10;
    Increment(ref myNumber);
    Console.WriteLine(myNumber); // 20 — changed by the method

    var p = new Person { Name = "Alice" };
    ReassignPerson(ref p);
    Console.WriteLine(p.Name); // "Bob" — reassignment affected the caller
}
```

**Rules for `ref`:**
- The variable **must be initialized** before being passed.
- Both the caller and the method signature must use the `ref` keyword.


## The `out` Keyword

The `out` keyword also passes by reference, but is designed for methods that **return additional values** through parameters. The caller does not need to initialize the variable, but the method **must assign a value** before returning.

```csharp
static bool TryParseDate(string input, out DateTime result)
{
    return DateTime.TryParse(input, out result);
}

static void Main()
{
    if (TryParseDate("2026-03-31", out DateTime date))
    {
        Console.WriteLine($"Parsed: {date:yyyy-MM-dd}"); // "Parsed: 2026-03-31"
    }
    else
    {
        Console.WriteLine("Invalid date.");
    }
}
```

**Rules for `out`:**
- The variable **does not need to be initialized** before the call.
- The called method **must assign** a value to the `out` parameter before it returns.
- Both caller and method must use the `out` keyword.

The `TryParse` pattern is the most common real-world usage of `out`.


## The `in` Keyword

The `in` keyword passes by reference but as **read-only**. The method cannot modify the value. This is primarily useful for large `struct` types to avoid the overhead of copying while guaranteeing the method won't mutate the data.

```csharp
readonly struct LargeData
{
    public readonly double X, Y, Z, W;

    public LargeData(double x, double y, double z, double w)
        => (X, Y, Z, W) = (x, y, z, w);
}

static double ComputeSum(in LargeData data)
{
    // data.X = 42; // Compile error: cannot modify 'in' parameter
    return data.X + data.Y + data.Z + data.W;
}

static void Main()
{
    var data = new LargeData(1.0, 2.0, 3.0, 4.0);
    Console.WriteLine(ComputeSum(in data)); // 10
}
```

**Rules for `in`:**
- The parameter is passed by reference but **cannot be modified** inside the method.
- The caller can optionally use the `in` keyword, but it's not required.
- Best suited for large value types where copying would be expensive.


## Summary

| Modifier | Must Initialize? | Method Can Modify? | Read-Only? | Typical Use Case |
|----------|------------------|--------------------|------------|------------------|
| *(default)* | Yes | Copy only (no effect on caller) | No | Standard parameter passing |
| `ref` | Yes | Yes (affects caller) | No | Swap methods, in-place modification |
| `out` | No | Must assign before return | No | `TryParse` patterns, multiple return values |
| `in` | Yes | No (compile error) | Yes | Large struct optimization |


## Comparison with Other Languages

### Python

Python uses a model often called **"pass by object reference"** or **"pass by assignment"**. There is no distinction between value types and reference types — everything is an object. However, whether mutations inside a function are visible to the caller depends on whether the object is **mutable** or **immutable**:

```python
# Immutable types (int, str, tuple) — reassignment creates a new object
def try_modify(x):
    x += 10  # creates a new int object, does NOT affect caller

value = 5
try_modify(value)
print(value)  # 5 — unchanged

# Mutable types (list, dict, set) — in-place modifications affect the caller
def try_modify_list(items):
    items.append(42)  # modifies the same list object

my_list = [1, 2, 3]
try_modify_list(my_list)
print(my_list)  # [1, 2, 3, 42] — changed
```

Python has **no equivalent** to C#'s `ref`, `out`, or `in` keywords. To return multiple values, Python uses tuple unpacking instead of `out` parameters:

```python
def try_parse_int(text):
    try:
        return True, int(text)
    except ValueError:
        return False, None

success, number = try_parse_int("42")
```

### Java

Java's model is very similar to C#'s default behavior: **everything is passed by value**. For primitive types (`int`, `double`, `boolean`) the value is copied. For objects, the reference is copied — so you can modify the object's fields, but reassigning the reference inside the method has no effect on the caller. Java has **no `ref` or `out` keywords**, so there is no way to pass by reference.

### Quick Comparison Table

| Feature | C# | Python | Java |
|---------|-----|--------|------|
| Default mechanism | Pass by value | Pass by object reference | Pass by value |
| Value types copied? | Yes | N/A (no value types) | Yes (primitives) |
| Object mutation visible? | Yes | Yes (if mutable) | Yes |
| Object reassignment visible? | No (unless `ref`) | No | No |
| Pass by reference support | `ref`, `out`, `in` | Not available | Not available |
| Multiple return values | `out`, tuples | Tuple unpacking | Return object/array |
