---
layout: page
title: NuGet - Package locations and config
parent: Other
---

# Package locations and config

The NuGet packages are stored in local caches on you computer. Solution-local packages folders are no longer used by .NET Core and Visual Studio. The command to list user-specific folders is:

```shell
dotnet nuget locals all --list
```

A typical output can be something like this:

```shell
http-cache: C:\Users\halletz\AppData\Local\NuGet\v3-cache
global-packages: C:\Users\halletz\.nuget\packages\
temp: C:\Users\halletz\AppData\Local\Temp\NuGetScratch
plugins-cache: C:\Users\halletz\AppData\Local\NuGet\plugins-cache
```

Notice that the machine-wide folder isn't listed with this. It is defined at Visual Studio settings: Options -> NuGet Package Manager -> Package Sources. By default it is:

`C:\Program Files (x86)\Microsoft SDKs\NuGetPackages\`

You can overwrite these cache locations in several NuGet config files.
NuGet.config files are located here:

```shell
User-specific: %APPDATA%\NuGet\
Machine-wide: %ProgramFiles(x86)%\NuGet\Config\
```

### Sources

[How NuGet settings are applied](https://learn.microsoft.com/en-us/nuget/consume-packages/configuring-nuget-behavior#how-settings-are-applied)
[NuGet global cache folders](https://learn.microsoft.com/en-us/nuget/consume-packages/managing-the-global-packages-and-cache-folders)