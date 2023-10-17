---
layout: page
title: C# - Preprocessor directives
parent: C#
---

# Preprocessor directives in C#

With preprocessor directives you can mark code to only be compiled in an assembly within a specified configuration mode of the assembly. 

If you want to debug a program it can come in handy to test certain functionality only within debugging. you could write a generated html file on disc, but you don't want that to happen in the release version of your program.

With a preprocessor directive that call to write the html file can be capsulated to only be compiled in the assembly in the selected debug configuration in Visual Studio:

[![VS Configuration](/assets/images/articles/preprocessor/vsconfig.png)](/assets/images/articles/preprocessor/vsconfig.png)

```csharp
#if DEBUG
    // Code here only in Debug Mode
#endif
```

To use and recognize this configuration, you have to define the DEBUG constant. There are two ways to get the definition active. You can either set it up in the csproj-file or activate it in the project settings GUI.

## .NET Framework

Define the constants with the project settings GUI:

[![Settings GUI](/assets/images/articles/preprocessor/constants-settings-gui.png)](/assets/images/articles/preprocessor/constants-settings-gui.png)


Define the constants with the csproj-file:

[![csproj](/assets/images/articles/preprocessor/constants-csproj.png)](/assets/images/articles/preprocessor/constants-csproj.png)


## .NET

In .NET 6 and newer the constants get defined with the project initialization. You can find them in the new project settings GUI aswell:

[![Settings GUI](/assets/images/articles/preprocessor/constants-settings-gui-new.png)](/assets/images/articles/preprocessor/constants-settings-gui-new.png)
