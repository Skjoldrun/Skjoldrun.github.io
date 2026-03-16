---
layout: page
title: C# - Inheritance & Interfaces
parent: C#
---

# C# Inheritance: Abstract Classes, Sealed Classes, and Interfaces

## Overview

Inheritance is one of the core concepts of object-oriented programming
(OOP).\
In **C#**, inheritance allows a class (called a *derived class*) to
reuse and extend the behavior of another class (called a *base class*).

This enables:

-   Code reuse
-   Logical modeling of relationships
-   Extensible architecture
-   Polymorphism

Understanding inheritance is essential for designing maintainable and
scalable applications.

------------------------------------------------------------------------

## Basic Class Inheritance

A class can inherit from another class using the `:` syntax.

### Example

``` csharp
// Base class
public class Animal
{
    public string Name { get; set; }

    public void Eat()
    {
        Console.WriteLine($"{Name} is eating.");
    }
}

// Derived class
public class Dog : Animal
{
    public void Bark()
    {
        Console.WriteLine($"{Name} is barking.");
    }
}
```

### Usage

``` csharp
Dog dog = new Dog();
dog.Name = "Rex";

dog.Eat();   // inherited from Animal
dog.Bark();  // defined in Dog
```

### Key Points

-   `Dog` **inherits** all public and protected members from `Animal`
-   The derived class can **extend functionality**
-   C# supports **single inheritance** (one base class only)

------------------------------------------------------------------------

## Method Overriding and Virtual Methods

Sometimes a derived class needs to **change the behavior** of a method
from the base class.

For that, C# provides:

-   `virtual`
-   `override`

### Example

``` csharp
public class Animal
{
    public virtual void MakeSound()
    {
        Console.WriteLine("The animal makes a sound.");
    }
}

public class Dog : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("The dog barks.");
    }
}
```

### Usage

``` csharp
Animal myAnimal = new Dog();

// Calls the Dog implementation due to polymorphism
myAnimal.MakeSound();
```

### Key Concept: Polymorphism

Even though the variable type is `Animal`, the runtime type (`Dog`)
determines which method is executed.

------------------------------------------------------------------------

## Abstract Classes

An **abstract class** represents a base class that **cannot be
instantiated directly**.

It is designed to be inherited by other classes.

Abstract classes can contain:

-   Abstract methods (no implementation)
-   Virtual methods
-   Fully implemented methods
-   Fields and properties

### Example

``` csharp
public abstract class Shape
{
    public string Name { get; set; }

    // Abstract method (must be implemented by derived classes)
    public abstract double CalculateArea();

    public void PrintName()
    {
        Console.WriteLine($"Shape: {Name}");
    }
}

public class Circle : Shape
{
    public double Radius { get; set; }

    public override double CalculateArea()
    {
        return Math.PI * Radius * Radius;
    }
}
```

### Usage

``` csharp
Circle circle = new Circle
{
    Name = "Circle",
    Radius = 5
};

Console.WriteLine(circle.CalculateArea());
```

### Important Rules

-   Abstract classes **cannot be instantiated**
-   Derived classes **must implement all abstract members**
-   Useful for **shared base behavior with required specialization**

------------------------------------------------------------------------

## Interfaces

An **interface** defines a **contract** that classes must implement.

Unlike abstract classes, interfaces:

-   Cannot contain fields
-   Cannot contain constructors
-   Only define members that must be implemented

Interfaces are commonly used to enforce **capabilities**.

### Example

``` csharp
public interface ILogger
{
    void Log(string message);
}

public class ConsoleLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine($"LOG: {message}");
    }
}
```

### Usage

``` csharp
ILogger logger = new ConsoleLogger();
logger.Log("Application started.");
```

### Multiple Interface Implementation

C# allows implementing **multiple interfaces**.

``` csharp
public interface IReadable
{
    void Read();
}

public interface IWritable
{
    void Write();
}

public class FileHandler : IReadable, IWritable
{
    public void Read()
    {
        Console.WriteLine("Reading file...");
    }

    public void Write()
    {
        Console.WriteLine("Writing file...");
    }
}
```

------------------------------------------------------------------------

## Sealed Classes

A **sealed class** prevents other classes from inheriting from it.

This is useful when:

-   The class is complete
-   You want to prevent extension
-   Security or architecture reasons require strict control

### Example

``` csharp
public sealed class ConfigurationManager
{
    public string GetSetting(string key)
    {
        // Example implementation
        return "SomeValue";
    }
}
```

Attempting to inherit from this class will cause a compile error:

``` csharp
// ❌ This will not compile
public class CustomConfigurationManager : ConfigurationManager
{
}
```

### When to Use Sealed

Common scenarios:

-   Utility classes
-   Performance-sensitive classes
-   Classes not designed for inheritance

------------------------------------------------------------------------

## Sealed Methods

You can also seal **overridden methods** to prevent further overrides.

``` csharp
public class BaseClass
{
    public virtual void Execute()
    {
        Console.WriteLine("Base execution.");
    }
}

public class DerivedClass : BaseClass
{
    public sealed override void Execute()
    {
        Console.WriteLine("Derived execution.");
    }
}
```

Now further derived classes **cannot override `Execute()` again**.

------------------------------------------------------------------------

## Abstract Class vs Interface

Feature                | Abstract Class    | Interface
-----------------------|-------------------|--------------------
Multiple inheritance   | No                | Yes
Fields allowed         | Yes               | No
Default implementation | Yes               | Yes (since C# 8)
Constructors           | Yes               | No
Intended usage         | Shared base logic | Capability contract

### Guideline

Use:

**Abstract Class when:**

-   You want shared implementation
-   You control the full hierarchy

**Interface when:**

-   Multiple unrelated classes should share behavior
-   You want loose coupling

------------------------------------------------------------------------

## Best Practices

### Prefer Composition over Inheritance

Inheritance can create **tight coupling**. Often it is better to compose
behavior.

Example:

``` csharp
public class Engine
{
    public void Start()
    {
        Console.WriteLine("Engine started.");
    }
}

public class Car
{
    private Engine _engine = new Engine();

    public void Start()
    {
        _engine.Start();
    }
}
```

### Keep Inheritance Hierarchies Shallow

Deep hierarchies make code hard to understand and maintain.

Recommended:

    BaseClass
       └ DerivedClass
           └ SpecializedClass

Avoid very deep trees.

### Use Interfaces for Flexibility

Interfaces enable:

-   Dependency Injection
-   Testing with mocks
-   Clean architecture

------------------------------------------------------------------------

## Real-World Example

A typical architecture might look like this:

``` csharp
public interface IRepository<T>
{
    T GetById(int id);
    void Save(T entity);
}

public abstract class RepositoryBase<T> : IRepository<T>
{
    public abstract T GetById(int id);

    public virtual void Save(T entity)
    {
        Console.WriteLine("Saving entity...");
    }
}

public class UserRepository : RepositoryBase<User>
{
    public override User GetById(int id)
    {
        return new User { Id = id };
    }
}

public class User
{
    public int Id { get; set; }
}
```

This pattern is common in:

-   ASP .NET applications
-   Clean Architecture
-   Domain-driven design

------------------------------------------------------------------------

# Summary

C# inheritance provides powerful tools for structuring code:

**Inheritance** - Reuse functionality from base classes

**Abstract Classes** - Define shared logic and required behavior

**Interfaces** - Define contracts and enable loose coupling

**Sealed Classes** - Prevent inheritance for stability and safety

Understanding when to use each mechanism is essential for building
clean, maintainable, and extensible software systems.
