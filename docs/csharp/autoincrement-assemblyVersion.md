---
layout: page
title: C# - Autoincrement AssemblyVersion
parent: C#
---

# Autoincrement AssemblyVersion

It can be usefull to have a kind of versioning logic for your assemblies. Analysing logs, comparing software assemblies in the wild or even having a history of versions will be easier with some kind of versioning.

A good approach is versioning with `Major.Minor.Patch` or with `Major.Minor.Build.Revision`.

You can automate the incrementing of parts of the version numbers in VS and dotNet and automate for example the Build and the Revision number.


## .NET Framework

In .NET Framework you had the possibility to automate this nativly with having some little configuration:

* Change the csproj-file with `<Deterministic>false</Deterministic>`
* Change the AssemblyInfo.cs file to have `*` for versioning in `[assembly: AssemblyVersion("1.0.*")]`

Changes in the csproj-file:
[![.NET Framework csproj](/assets/images/coding/csharp/autoincrement-assemblyVersion/framework-csproj.png)](/assets/images/coding/csharp/autoincrement-assemblyVersion/framework-csproj.png)

Changes in AssemblyInfo.cs:
[![.NET Framework AssemblyInfo.cs](/assets/images/coding/csharp/autoincrement-assemblyVersion/framework-assemblyInfo.png)](/assets/images/coding/csharp/autoincrement-assemblyVersion/framework-assemblyInfo.png)


## .NET

With .NET there are multiple ways to get autoincremented Version numbers:

* the "old" .NET Framework way, like above
* the new csproj-file in sdk style way with major enhancements


### The old way

In .NET there is no AssemblyInfo.cs visible in the project by default. The file gets autocreated and it resides in the obj folder for the targeted environment of the project. But you can still use the above .NET Framework solution with some preparation. With deactivating the autogeneration of the AssemblyInfo.cs file in the csporj-file, you can create the file for yourself and write the same entries as in the .NET Framework file.

Dactivating GenerateAssemblyInfo in csproj:
```xml
<PropertyGroup>
   <GenerateAssemblyInfo>false</GenerateAssemblyInfo>
</PropertyGroup>
```

Create and fill the File with a properties folder in the project:

[![.NET create AssemblyInfo.cs](/assets/images/coding/csharp/autoincrement-assemblyVersion/dotnet-create-file.png)](/assets/images/coding/csharp/autoincrement-assemblyVersion/dotnet-create-file.png)


### The better way

With the csproj-file in [sdk style](https://docs.microsoft.com/en-us/dotnet/core/project-sdk/overview) you can specify the same values as in the former assemblyInfo.cs, but these values are also used for NuGet packages. This makes versioning for your assemblies much easier because you don't have to do it multiple times any more. On top of that, you now have the possibilities to enhance your versioning with own logic for generating version numbers.

The following example shows versioning with `Major.Minor.Build.Revision`, but the build number is the mont and day as `MMdd` and the revision number is the hour and minute as `HHmm` of the build time:

```xml
<PropertyGroup>
	<AssemblyName>YourAssemblyName</AssemblyName>
	<RootNamespace>YourAssemblyNameSpace</RootNamespace>
	<VersionSuffix>1.0.$([System.DateTime]::UtcNow.ToString(MMdd)).$([System.DateTime]::Now.ToString(HHmm))</VersionSuffix>
	<AssemblyVersion Condition=" '$(VersionSuffix)' == '' ">0.0.0.1</AssemblyVersion>
	<AssemblyVersion Condition=" '$(VersionSuffix)' != '' ">$(VersionSuffix)</AssemblyVersion>
	<Version Condition=" '$(VersionSuffix)' == '' ">0.0.1.0</Version>
	<Version Condition=" '$(VersionSuffix)' != '' ">$(VersionSuffix)</Version>
	<Company>YourCompany</Company>
	<Authors>YourName</Authors>
	<Copyright>Copyright © $(Company) $([System.DateTime]::UtcNow.ToString(yyyy))</Copyright>
	<Product>YourProductName</Product>
	<Description>Some description.</Description>
	<GeneratePackageOnBuild>False</GeneratePackageOnBuild>
</PropertyGroup>
```

[![.NET Version example](/assets/images/coding/csharp/autoincrement-assemblyVersion/dotnet-version-number.png)](/assets/images/coding/csharp/autoincrement-assemblyVersion/dotnet-version-number.png)