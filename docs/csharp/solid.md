---
layout: page
title: C# - SOLID principles
parent: C#
---

# The SOLID Principles — A Star Wars Guide to Clean Code

> The **SOLID** principles are five fundamental guidelines that help you design software that is easy to understand, extend, and maintain. A solid foundation makes all the difference.

---

## Table of Contents

1. [What is SOLID?](#what-is-solid)
2. [S — Single Responsibility Principle (SRP)](#s--single-responsibility-principle-srp)
3. [O — Open/Closed Principle (OCP)](#o--openclosed-principle-ocp)
4. [L — Liskov Substitution Principle (LSP)](#l--liskov-substitution-principle-lsp)
5. [I — Interface Segregation Principle (ISP)](#i--interface-segregation-principle-isp)
6. [D — Dependency Inversion Principle (DIP)](#d--dependency-inversion-principle-dip)
7. [Putting It All Together](#putting-it-all-together)
8. [Exercises](#exercises)

---

## What is SOLID?

SOLID is an acronym coined by Robert C. Martin (Uncle Bob) that represents five design principles for writing clean, flexible, and maintainable object-oriented code. Think of these principles as the **Jedi Code** of software engineering — they guide you toward the Light Side of development and help you resist the temptation of quick hacks (the Dark Side).

| Letter | Principle | In Star Wars Terms |
|--------|-----------|-------------------|
| **S** | Single Responsibility | A Jedi has one master, one mission |
| **O** | Open/Closed | The Force grows — it doesn't replace |
| **L** | Liskov Substitution | Any Jedi can take another's place on a mission |
| **I** | Interface Segregation | A Pilot doesn't need a lightsaber |
| **D** | Dependency Inversion | The Jedi Council depends on the Force, not individual Jedi |

---

## S — Single Responsibility Principle (SRP)

> **A class should have one, and only one, reason to change.**

Imagine R2-D2. He's an astromech droid — he repairs ships, stores messages, and interfaces with computers. Now imagine if R2-D2 was also responsible for cooking meals, flying the Millennium Falcon, *and* commanding the Rebel fleet. That would be one overloaded droid! If any one of those responsibilities changed, you'd have to rebuild the entire droid.

The **Single Responsibility Principle** says: give each class exactly **one job**. If a class does too many things, split it up.

### ❌ Bad Example — The "Death Star" Class

```csharp
namespace SOLIDEDU.ConsoleUI
{
    // This class does WAY too much — it's the Death Star of code.
    public class StarWarsGame
    {
        public void CreateCharacter(string name, string faction)
        {
            Console.WriteLine($"Creating character: {name} of {faction}");
        }

        public void SaveToFile(string name)
        {
            File.WriteAllText($"{name}.txt", $"Character: {name}");
        }

        public void PrintReport(string name)
        {
            Console.WriteLine($"=== Report for {name} ===");
        }

        public void SendNotification(string message)
        {
            Console.WriteLine($"Notification: {message}");
        }
    }
}
```

This class handles character creation, file saving, reporting, *and* notifications. If the way we save files changes, we also have to touch the class that creates characters. These things have nothing to do with each other.

### ✅ Good Example — Separate Responsibilities

```csharp
namespace SOLIDEDU.ConsoleUI
{
    public class CharacterFactory
    {
        public Character Create(string name, string faction)
        {
            return new Character { Name = name, Faction = faction };
        }
    }

    public class CharacterRepository
    {
        public void Save(Character character)
        {
            File.WriteAllText($"{character.Name}.txt", $"Character: {character.Name}");
        }
    }

    public class CharacterReporter
    {
        public void PrintReport(Character character)
        {
            Console.WriteLine($"=== Report for {character.Name} ===");
            Console.WriteLine($"Faction: {character.Faction}");
        }
    }

    public class Character
    {
        public string Name { get; set; }
        public string Faction { get; set; }
    }
}
```

Now each class has **one reason to change**. The factory creates characters. The repository handles persistence. The reporter handles display logic. Clean, simple, and maintainable — just like a well-organized Rebel base.

---

## O — Open/Closed Principle (OCP)

> **Software entities should be open for extension, but closed for modification.**

Think of a lightsaber. The core mechanism stays the same, but you can swap out the Kyber crystal to change its color and properties. You **extend** the lightsaber's capabilities without tearing apart its internal design.

The **Open/Closed Principle** says: you should be able to add new behavior to your system **without changing existing code**. This is typically achieved through **abstraction** (interfaces, abstract classes) and **polymorphism**.

### ❌ Bad Example — Modifying Existing Code for Every New Type

```csharp
namespace SOLIDEDU.ConsoleUI
{
    public class AttackCalculator
    {
        public int CalculateDamage(string weaponType)
        {
            // Every time we add a new weapon, we MODIFY this method.
            if (weaponType == "Lightsaber")
                return 100;
            else if (weaponType == "Blaster")
                return 50;
            else if (weaponType == "Bowcaster")
                return 75;
            // What about a thermal detonator? We'd have to change this code again...
            else
                return 10;
        }
    }
}
```

Every new weapon type means modifying `CalculateDamage`. This risks breaking existing logic every time.

### ✅ Good Example — Open for Extension via Abstraction

```csharp
namespace SOLIDEDU.ConsoleUI
{
    public interface IWeapon
    {
        string Name { get; }
        int CalculateDamage();
    }

    public class Lightsaber : IWeapon
    {
        public string Name => "Lightsaber";
        public int CalculateDamage() => 100;
    }

    public class Blaster : IWeapon
    {
        public string Name => "Blaster";
        public int CalculateDamage() => 50;
    }

    public class Bowcaster : IWeapon
    {
        public string Name => "Bowcaster";
        public int CalculateDamage() => 75;
    }

    // Adding a new weapon? Just create a new class — no existing code changes!
    public class ThermalDetonator : IWeapon
    {
        public string Name => "Thermal Detonator";
        public int CalculateDamage() => 200;
    }

    public class AttackCalculator
    {
        public int CalculateDamage(IWeapon weapon)
        {
            return weapon.CalculateDamage();
        }
    }
}
```

Now you can add a `ThermalDetonator`, a `ForceLightning` attack, or any new weapon **without touching** `AttackCalculator`. The system is **open** for extension and **closed** for modification.

---

## L — Liskov Substitution Principle (LSP)

> **Objects of a superclass should be replaceable with objects of a subclass without breaking the application.**

If the Jedi Council sends a Jedi on a mission, it shouldn't matter whether that Jedi is Obi-Wan Kenobi, Anakin Skywalker, or Ahsoka Tano. Any Jedi should be able to fulfill the role. But if you send a youngling who can't yet use the Force properly, the mission falls apart — that's a Liskov violation.

The **Liskov Substitution Principle** says: if class `B` is a subclass of class `A`, then you should be able to use `B` anywhere `A` is expected, **without surprises**.

### ❌ Bad Example — A Subclass That Breaks the Contract

```csharp
namespace SOLIDEDU.ConsoleUI
{
    public class Droid
    {
        public virtual string Speak()
        {
            return "Beep boop!";
        }

        public virtual void Hack(string system)
        {
            Console.WriteLine($"Hacking into {system}...");
        }
    }

    public class ProtocolDroid : Droid
    {
        public override string Speak()
        {
            return "I am fluent in over six million forms of communication.";
        }

        public override void Hack(string system)
        {
            // Protocol droids can't hack! This breaks the contract.
            throw new NotSupportedException("Protocol droids cannot hack systems!");
        }
    }
}
```

Code that works with a `Droid` expects `Hack()` to work. Substituting a `ProtocolDroid` causes a crash. The subclass breaks the promise of the base class.

### ✅ Good Example — Proper Hierarchy

```csharp
namespace SOLIDEDU.ConsoleUI
{
    public abstract class Droid
    {
        public abstract string Speak();
    }

    public interface IHackable
    {
        void Hack(string system);
    }

    public class AstromechDroid : Droid, IHackable
    {
        public override string Speak() => "Beep boop!";

        public void Hack(string system)
        {
            Console.WriteLine($"Hacking into {system}...");
        }
    }

    public class ProtocolDroid : Droid
    {
        public override string Speak() => "I am fluent in over six million forms of communication.";
        // No Hack method — protocol droids simply don't have that capability.
    }
}
```

Now `ProtocolDroid` no longer pretends it can hack. Both droids can `Speak()` (the shared contract), but only `AstromechDroid` implements `IHackable`. You can substitute any `Droid` wherever a `Droid` is expected, and nothing breaks.

---

## I — Interface Segregation Principle (ISP)

> **No client should be forced to depend on methods it does not use.**

Imagine a Stormtrooper. Stormtroopers shoot blasters — that's their thing. Now imagine forcing every Stormtrooper to also implement `UseTheForce()` and `BuildLightsaber()`. That's absurd — those are Jedi skills, not Stormtrooper skills.

The **Interface Segregation Principle** says: don't create bloated interfaces. Instead, create **small, focused interfaces** that each describe a specific capability.

### ❌ Bad Example — One Fat Interface

```csharp
namespace SOLIDEDU.ConsoleUI
{
    public interface IGalacticWarrior
    {
        void ShootBlaster();
        void UseTheForce();
        void BuildLightsaber();
        void PilotStarship();
        void SliceComputer();
    }

    // A Stormtrooper is forced to "implement" things it can't do.
    public class Stormtrooper : IGalacticWarrior
    {
        public void ShootBlaster() => Console.WriteLine("Pew pew! (Missed again)");
        public void UseTheForce() => throw new NotSupportedException();
        public void BuildLightsaber() => throw new NotSupportedException();
        public void PilotStarship() => Console.WriteLine("Piloting TIE Fighter");
        public void SliceComputer() => throw new NotSupportedException();
    }
}
```

Three out of five methods throw exceptions. The interface demands too much.

### ✅ Good Example — Segregated Interfaces

```csharp
namespace SOLIDEDU.ConsoleUI
{
    public interface IShooter
    {
        void ShootBlaster();
    }

    public interface IForceUser
    {
        void UseTheForce();
        void BuildLightsaber();
    }

    public interface IPilot
    {
        void PilotStarship();
    }

    public interface ISlicer
    {
        void SliceComputer();
    }

    public class Stormtrooper : IShooter, IPilot
    {
        public void ShootBlaster() => Console.WriteLine("Pew pew!");
        public void PilotStarship() => Console.WriteLine("Piloting TIE Fighter");
    }

    public class Jedi : IForceUser, IPilot
    {
        public void UseTheForce() => Console.WriteLine("Using the Force...");
        public void BuildLightsaber() => Console.WriteLine("Crafting a lightsaber...");
        public void PilotStarship() => Console.WriteLine("Piloting X-Wing");
    }
}
```

Now each class only implements what it actually does. A `Stormtrooper` shoots and pilots. A `Jedi` uses the Force and pilots. No one is forced to implement capabilities they don't have.

---

## D — Dependency Inversion Principle (DIP)

> **High-level modules should not depend on low-level modules. Both should depend on abstractions.**

The Jedi Council doesn't directly manage every lightsaber crystal mine or every starship hangar. Instead, it depends on **abstractions** — it issues orders and trusts that the lower-level systems fulfill them. If the Council had to know the exact internal workings of every mine, changing one mine's process would require changing the Council itself.

The **Dependency Inversion Principle** says: depend on **interfaces** (abstractions), not on **concrete implementations**. This makes your code flexible and testable.

### ❌ Bad Example — High-Level Depends on Low-Level

```csharp
namespace SOLIDEDU.ConsoleUI
{
    public class HoloNet
    {
        public void SendMessage(string message)
        {
            Console.WriteLine($"[HoloNet] {message}");
        }
    }

    public class JediCouncil
    {
        // Directly depends on HoloNet — a concrete class.
        private HoloNet _holoNet = new HoloNet();

        public void IssueOrder(string order)
        {
            _holoNet.SendMessage($"Council Order: {order}");
        }
    }
}
```

`JediCouncil` is tightly coupled to `HoloNet`. Want to switch to a different communication system? You'd have to modify `JediCouncil`.

### ✅ Good Example — Depend on Abstractions

```csharp
namespace SOLIDEDU.ConsoleUI
{
    public interface IMessageService
    {
        void SendMessage(string message);
    }

    public class HoloNet : IMessageService
    {
        public void SendMessage(string message)
        {
            Console.WriteLine($"[HoloNet] {message}");
        }
    }

    public class ForceTelepathy : IMessageService
    {
        public void SendMessage(string message)
        {
            Console.WriteLine($"[Force Telepathy] {message}");
        }
    }

    public class JediCouncil
    {
        private readonly IMessageService _messageService;

        // The dependency is INJECTED — the Council doesn't create it.
        public JediCouncil(IMessageService messageService)
        {
            _messageService = messageService;
        }

        public void IssueOrder(string order)
        {
            _messageService.SendMessage($"Council Order: {order}");
        }
    }
}
```

Now `JediCouncil` depends on `IMessageService`, not on `HoloNet`. You can swap in `ForceTelepathy`, a `DroidMessenger`, or a `MockMessageService` for testing — all without changing a single line of `JediCouncil`.

---

## Putting It All Together

The SOLID principles work best **together**. Here's a quick summary:

| Principle | Rule of Thumb | Star Wars Analogy |
|-----------|--------------|-------------------|
| **SRP** | One class = one job | R2-D2 repairs, C-3PO translates — they don't swap roles |
| **OCP** | Extend, don't modify | Add a new Kyber crystal, don't redesign the lightsaber |
| **LSP** | Subclasses must honor the base contract | Any Jedi can take a mission meant for a Jedi |
| **ISP** | Small, focused interfaces | Stormtroopers don't need Force powers in their job description |
| **DIP** | Depend on abstractions | The Council issues orders through a communication interface, not a specific device |

> *"Your focus determines your reality."* — Qui-Gon Jinn
>
> Focus on writing clean, principled code, and your software will be resilient, flexible, and ready to scale — just like the Rebel Alliance.
