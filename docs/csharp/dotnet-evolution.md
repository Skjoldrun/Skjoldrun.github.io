---
layout: page
title: C# - dotnet Evolution
parent: C#
---

# dotnet Evolution - Understanding .NET Framework, .NET Standard, and Modern .NET

## 1. The Beginning: .NET Framework

-   **Release year**: 2002
-   **Platform**: Windows only
-   **Primary use case**: Desktop applications (WinForms, WPF), ASP.NET
    web applications, enterprise solutions
-   **Characteristics**:
    -   Large and mature library ecosystem
    -   Tight integration with Windows (registry, COM, IIS)
    -   Monolithic: you depend on what Microsoft ships with Windows
        updates

The .NET Framework was revolutionary at its time, unifying programming
across languages (C#, VB.NET, F#). However, it was tied to Windows and
could not easily adapt to cross-platform needs.

---

## 2. The Bridge: .NET Standard

-   **Release year**: 2016
-   **Platform**: Not a runtime, but a **specification**
-   **Purpose**: To provide a common base API set that all .NET
    implementations must support
-   **Example**: If you target **.NET Standard 2.0**, your library can
    be used in both .NET Framework, .NET Core, Xamarin, and later .NET
    5+

This was especially important for **class libraries**, since developers
wanted their code to run across different runtimes without re-writing.\
Think of **.NET Standard as a contract**: "if my library requires these
APIs, then any runtime that implements this standard can run it."

---

## 3. The Modern Era: .NET Core and Beyond

-   **Release year**: 2016 (.NET Core 1.0)
-   **Cross-platform**: Runs on Windows, Linux, macOS
-   **Open source**: Fully developed in the open on GitHub
-   **Modular**: Smaller, faster, with NuGet-based libraries instead of
    being monolithic

Key improvements over .NET Framework:
- **Performance**: Much faster runtime and JIT optimizations
- **Cross-platform development**: Cloud-native and container-friendly
- **Side-by-side installs**: Different projects can run different
runtime versions on the same machine
- **Unified development**: From desktop apps to web apps, microservices,
and cloud workloads

In 2020, Microsoft introduced **.NET 5**, which dropped the "Core"
branding and unified all future development under just **.NET**. This
continues today with versions like **.NET 6, .NET 7, .NET 8**, and
upcoming releases.

---

## 4. What Should a New Programmer Know?

-   **.NET Framework**: Legacy only. Use it only if maintaining old
    enterprise applications.
-   **.NET Standard**: Useful if you are writing libraries that must run
    on both old and new runtimes.
-   **Modern .NET (Core and newer)**: The recommended choice for all
    **new projects**.

### Simple Rule of Thumb

-   If you are starting **today** → use **.NET 8+**
-   If you are building **libraries for maximum compatibility** → target
    **.NET Standard 2.0**
-   If you are maintaining **existing enterprise software** → stay with
    **.NET Framework** but plan migration

---

## 5. Summary Table

  
Feature       | .NET Framework   | .NET Standard           | .NET (Core & newer)
--------------|------------------|-------------------------|-----------------------------------
First Release | 2002             | 2016                    | 2016
Platform      | Windows only     | Specification only      | Windows, Linux, macOS
Open Source   | No               | N/A                     | Yes
Use Case      | Legacy apps      | Shared libraries        | Modern apps, cloud, cross-platform
Status Today  | Maintenance mode | Still relevant for libs | Actively developed

---

## 6. Final Thoughts

As a developer, the **direction is clear**:
Microsoft and the community are investing in **modern .NET**.

-   **Learn .NET 8+** if you are new.
-   **Understand .NET Framework** only if you work in companies with
    legacy systems.
-   **Know .NET Standard** if you plan to create reusable libraries.

In practice, you will mostly work with **modern .NET**, and that's where
the future lies.
