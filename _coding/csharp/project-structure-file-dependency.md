---
layout: page
title: C# - Project file dependency
---

# Project structure file dependency

If you want to show a dependency of two separate files in the Visual Studio project structure, you can use a definition of a dependency between the two files in the csproj file.


**Before:**

![Before](/assets/images/coding/csharp/project-structure-file-dependency/before.png)

Set this snippet into your csproj file to create the dependency of the desired files:

```xml
  <ItemGroup>
	  <None Update="appsettings.Development.json">
		  <DependentUpon>appsettings.json</DependentUpon>
	  </None>
  </ItemGroup>
```


**After:**

![After](/assets/images/coding/csharp/project-structure-file-dependency/after.gif)
