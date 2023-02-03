---
layout: page
title: C# - Project .props & .targets
parent: C#
---

# Project .props and .targets files

The .props and .targets files are MSBuild configuration files that get imported for the project, where they are places, or by all projects if these file are placed in a parent folder, e.g. on solution file level. 

The `Directory.Build.props` and the `Directory.Build.targets` files may then contain property and item groups to define your own settings, or to centralize these over multiple projects. The .props files get loaded from the MSBuild system at an early stage of the build and the .targets file get loaded in the end. You could update items in the .targets file that have been defined before, but only if they already exist.


```batch
|
|   Directory.Build.props
|   Directory.Build.targets
|   VersioningEDU.sln
|
+---VersioningEDU.ConsoleUI
|   |   Program.cs
|   |   VersioningEDU.ConsoleUI.csproj
|
\---VersioningEDU.Lib
    |   AssemblyInfoProcessor.cs
    |   VersioningEDU.Lib.csproj
|
```

## Multi Project centralized versioning

You can have a centralized versioning with these files with adding these at solution file level, or outside your listed project directories. Both filetypes get imported automatically and can hold centralized project properties and items to configure common values like versions, author, product name, copyright and more.

**csproj file:**
```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net7.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

	<PropertyGroup>
		<AssemblyName>VersioningEDU.ConsoleUI</AssemblyName>
		<RootNamespace>VersioningEDU.ConsoleUI</RootNamespace>
		<Description>Project to test dynamic Versioning.</Description>
		<GeneratePackageOnBuild>False</GeneratePackageOnBuild>
		<!-- Versioning and other Properties are centralized in Directory.Build.props -->
	</PropertyGroup>

  <ItemGroup>
    <ProjectReference Include="..\VersioningEDU.Lib\VersioningEDU.Lib.csproj" />
  </ItemGroup>

</Project>
```

**Directory.Build.props file**
```xml
<Project>
	<PropertyGroup>
		<VersionSuffix>1.0.$([System.DateTime]::UtcNow.ToString(MMdd)).$([System.DateTime]::Now.ToString(HHmm))</VersionSuffix>
		<AssemblyVersion Condition=" '$(VersionSuffix)' == '' ">0.0.0.1</AssemblyVersion>
		<AssemblyVersion Condition=" '$(VersionSuffix)' != '' ">$(VersionSuffix)</AssemblyVersion>
		<Version Condition=" '$(VersionSuffix)' == '' ">0.0.1.0</Version>
		<Version Condition=" '$(VersionSuffix)' != '' ">$(VersionSuffix)</Version>
		<Company>Hermann Otto Chemie</Company>
		<Authors>David T. Halletz</Authors>
		<Copyright>Copyright © $(Company) $([System.DateTime]::UtcNow.ToString(yyyy))</Copyright>
		<Product>VersioningEDU</Product>
	</PropertyGroup>
</Project>
```

Now every project will import those values and generate AssemblyInfo.cs classes with them.
