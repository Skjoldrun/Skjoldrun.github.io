---
layout: page
title: C# - App BaseDirectory comparison
parent: C#
---

# App BaseDirectory comparison: Understanding Path Resolution in .NET: `GetCurrentDirectory`, `GetEntryAssembly`, and `AppContext`

When working with configuration files, resources, or plugins in .NET applications, you often need to resolve file system paths. However, depending on whether your code runs in an **EXE** or a **DLL**, the results can vary. In this article, we compare the most common approaches.

---

## `Directory.GetCurrentDirectory()`

This method returns the **current working directory of the process**. It is **not guaranteed** to be the same as the application directory.

```csharp
using System;
using System.IO;

Console.WriteLine("Current Directory:");
Console.WriteLine(Directory.GetCurrentDirectory());
```

**Example output:**
```
C:\Temp
```

If you start your app from `C:\Temp`, this will return `C:\Temp`, even if your app is actually located in `C:\Projects\MyApp\bin\Debug\net8.0`.

✅ Good for CLI tools when you want the context of the user’s working folder.  
❌ Not safe for locating application resources (services may start in `C:\Windows\System32`).

---

## `Assembly.GetEntryAssembly().Location`

This method returns the **path to the entry assembly** (the main EXE or the DLL you run with `dotnet MyApp.dll`). It always points to the host program, not the library.

```csharp
using System;
using System.Reflection;

Console.WriteLine("Entry Assembly Location:");
Console.WriteLine(Assembly.GetEntryAssembly()?.Location);
```

**Example output:**
```
C:\Projects\MyApp\bin\Debug\net8.0\MyApp.dll
```

✅ Reliable when you want the application’s executable path.  
❌ Can be `null` in unit tests or unmanaged hosting scenarios.

---

## `AppContext.BaseDirectory`

This method returns the **base directory of the application domain**. In practice, this is usually the folder where the main EXE resides.

```csharp
using System;

Console.WriteLine("AppContext BaseDirectory:");
Console.WriteLine(AppContext.BaseDirectory);
```

**Example output:**
```
C:\Projects\MyApp\bin\Debug\net8.0\
```

✅ Always non-null.  
✅ Works consistently across Console apps, Web apps, Services, and Tests.  
👉 Often the best choice for loading application resources.

---

## `Assembly.GetExecutingAssembly().Location`

If you specifically want the **path of the DLL containing the currently executing code**, use this method.

```csharp
using System;
using System.Reflection;

Console.WriteLine("Executing Assembly Location:");
Console.WriteLine(Assembly.GetExecutingAssembly().Location);
```

**Example output (from MyLibrary.dll):**
```
C:\Projects\MyApp\bin\Debug\net8.0\MyLibrary.dll
```

✅ Useful when a library carries its own resources.  
❌ Not the same as the app’s base path.

---

## Comparison Table

| Method                                     | Returns                             | Notes                                     |
|--------------------------------------------|-------------------------------------|-------------------------------------------|
| `Directory.GetCurrentDirectory()`          | Process working directory           | Depends on where the app was launched     |
| `Assembly.GetEntryAssembly().Location`     | Path of the entry EXE/DLL           | Can be `null` in test/unmanaged scenarios |
| `AppContext.BaseDirectory`                 | Application base folder             | Safe, reliable, always available          |
| `Assembly.GetExecutingAssembly().Location` | Path of the currently executing DLL | Useful for library-specific resources     |

---

## Recommendation
- Use **`AppContext.BaseDirectory`** for application configs and resources.  
- Use **`Assembly.GetExecutingAssembly().Location`** for library-specific resources.  
- Avoid relying on **`Directory.GetCurrentDirectory()`** unless you explicitly need the user’s working directory.
