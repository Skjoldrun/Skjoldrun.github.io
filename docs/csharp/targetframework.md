---
layout: page
title: C# - Targetframework combinations
parent: C#
---

# Targetframework combinations

Changing from a full .NET Framework 4.8 Stack to Core and newer isn't always that easy. It might be necessary to have both worlds in parallel for a while. Luckily you have some possibilities to get this done for your projects.


## .NET Project Files with TargetFrameworks

In the .NET Core and newer project files exists the `TargetFramework[s]` tags to set the framework to be compiled to. Setting this up with the TargetFrameworks of `<TargetFrameworks>net6.0; net48</TargetFrameworks>` generates output for both worlds, if the codebase is compatible. That means it has to be written in the minor Framework compatibility of .NET Framework 4.8. You can't use features like global usings, or shorter Namespace definitions without the `{}` blocks. The .NET Core and newer is backward compatible to the .NET Framework 4.8 coding styles, so you have to choose this as basis.

**Example Project:**

```xml
<Project Sdk="Microsoft.NET.Sdk">

	<PropertyGroup>
		<OutputType>Exe</OutputType>
		<TargetFrameworks>net6.0; net48</TargetFrameworks>
		<ImplicitUsings>disable</ImplicitUsings>
		<Nullable>disable</Nullable>
		<AutoGenerateBindingRedirects>True</AutoGenerateBindingRedirects>
	</PropertyGroup>

	<PropertyGroup>
		<AssemblyName>TargetFrameworkEDU.UI</AssemblyName>
		<RootNamespace>TargetFrameworkEDU.UI</RootNamespace>
		<VersionSuffix>1.0.$([System.DateTime]::UtcNow.ToString(MMdd)).$([System.DateTime]::Now.ToString(HHmm))</VersionSuffix>
		<AssemblyVersion Condition=" '$(VersionSuffix)' == '' ">0.0.0.1</AssemblyVersion>
		<AssemblyVersion Condition=" '$(VersionSuffix)' != '' ">$(VersionSuffix)</AssemblyVersion>
		<Version Condition=" '$(VersionSuffix)' == '' ">0.0.1.0</Version>
		<Version Condition=" '$(VersionSuffix)' != '' ">$(VersionSuffix)</Version>
		<Company>My Company</Company>
		<Authors>David T. Halletz</Authors>
		<Copyright>Copyright © $(Company) $([System.DateTime]::UtcNow.ToString(yyyy))</Copyright>
		<Product>TargetFrameworkEDU</Product>
		<Description>Test Application to try to work with different targetframeworks.</Description>
		<GeneratePackageOnBuild>False</GeneratePackageOnBuild>
	</PropertyGroup>

	<ItemGroup>
	  <ProjectReference Include="..\TargetFrameworkEDU.FrameworkLIB\TargetFrameworkEDU.FrameworkLIB.csproj" />
	  <ProjectReference Include="..\TargetFrameworkEDU.LIB\TargetFrameworkEDU.LIB.csproj" />
	  <ProjectReference Include="..\TargetFrameworkEDU.NETLIB\TargetFrameworkEDU.NETLIB.csproj" />
	</ItemGroup>

</Project>
```

As you can see, I have already tested the integration of different Libraries. One is with both framework targets as well. Another one is a .NET Framework 4.8 ClassLibrary project. A third one is a .NET 6.0 ClassLibrary project. Having the main project reference all of these, shows what is possible to build and the compatibilities between them:

| Main Project | Referenced Project | Compatibility |
| --- | --- | --- |
| net6.0; net48 | net6.0; net48 | yes, with [.NET Framework compatibility mode](https://learn.microsoft.com/en-us/dotnet/core/porting/#net-framework-compatibility-mode) |
| net6.0; net48 | net48 | yes, with [.NET Framework compatibility mode](https://learn.microsoft.com/en-us/dotnet/core/porting/#net-framework-compatibility-mode) |
| net6.0; net48 | net6.0 | no, net48 cannot load the net6.0 library by default |
| net48 / Framework 4.8 | net6.0; net48 | yes, but only with the net48 target |

The [.NET Framework compatibility mode](https://learn.microsoft.com/en-us/dotnet/core/porting/#net-framework-compatibility-mode) gives access to .NET Framework 4.8 libraries on Windows only and then only in special cases. The references library can then just contain features that can be called from .NET Core and newer. 


## .NET and .NET Framework parallel Project Files and Solutions

Another possibility would be to use two separate project and solution files for the TargetFrameworks. You then have to create solutions  and internal project files for each Targetframework wich only uses its certain target, but both include the same code files. The code files then have to be compatible with each TargetFramework must be written on the lower framework and C# language level.

This approach has the flaw, that you always have to be sure to not alter the base coding files in a way to break the other solution and project files. Keeping both in sync could be really hard.


# Conclusion

From the perspective of a developer who is new to this kind of parallel work, it seems to be the better choice to use the .NET Core and newer project files with multiple TargetFrameworks than the parallel solution and project files.

This might also change in the future with having more experience with this topic. 