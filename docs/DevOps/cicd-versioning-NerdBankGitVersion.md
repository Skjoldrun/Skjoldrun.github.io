---
layout: page
title: DevOps - Versioning with NerdBank.GitVersion
parent: DevOps
---

# Versioning with NerdBank.GitVersion

I formerly tested the Versioning tool [MinVer](https://github.com/adamralph/minver) but was not satisfied for all my CI/CD needs. Therefore I tried the tool [NerdBank.GitVersion](https://github.com/dotnet/Nerdbank.GitVersioning) and succeeded with implementing a minimalistic, configurable versioning with DevOps integration and access to the version numbers in the Buildpipe and the Releasepipe as well.


## Installation

Run the following commands with your dotnet CLI to trigger the installation of the tool:

```batch
dotnet tool install -g nbgv
```

Then install the package in your repo with:

```batch
nbgv install
```


## .NET Assemblies

The package integrates in the build process and stamps the version number into the assemblies. It works with [Semantic Versioning](https://semver.org/spec/v2.0.0.html) and uses the Major and Minor numbers for the assemblyVersion, a git height calculated Patch number and can use a git hash or a calculated integer from the git hash as Revision number. The tool would work with prerelease version suffixes, too.

```csharp
[assembly: System.Reflection.AssemblyVersion("1.0")]
[assembly: System.Reflection.AssemblyFileVersion("1.0.24.15136")]
[assembly: System.Reflection.AssemblyInformationalVersion("1.0.24-alpha+g9a7eb6c819")]
```


### What is the 'git height'?
Git 'height' is the number of commits in the longest path from HEAD (the code you're building) to some origin point, inclusive. In this case the origin is the commit that set the major.minor version number to the values found in the HEAD.

For example, if the version specified at HEAD is 1.1 and the longest path in git history from HEAD to where the version file was changed to 1.1 includes 15 commits, then the git height is "15" and the version would look something like this: `1.1.15`.


## ThisAssembly Class

NBGV injects an `internal sealed partial class ThisAssembly` into your project at build time. This class provides compile-time `const` and `static readonly` access to version and assembly metadata — no need to use reflection at runtime.

### Generated Members

The full set of generated members depends on your project settings and `version.json` configuration:

```csharp
internal sealed partial class ThisAssembly {
    // Version strings
    internal const string AssemblyVersion = "1.0";
    internal const string AssemblyFileVersion = "1.0.24.15136";
    internal const string AssemblyInformationalVersion = "1.0.24-alpha+g9a7eb6c819";

    // Assembly metadata
    internal const string AssemblyName = "MyProject";
    internal const string AssemblyTitle = "MyProject";
    internal const string AssemblyProduct = "MyProduct";
    internal const string AssemblyCompany = "MyCompany";
    internal const string AssemblyCopyright = "Copyright © 2026";
    internal const string AssemblyConfiguration = "Debug";
    internal const string RootNamespace = "MyProject";

    // Git information
    internal const string GitCommitId = "9a7eb6c8197c3600a5226038811ab3a869f726e2";
    internal static readonly System.DateTime GitCommitDate = new System.DateTime(638789145600000000L, System.DateTimeKind.Utc);

    // Release flags
    internal const bool IsPublicRelease = false;
    internal const bool IsPrerelease = true;

    // Strong name key info (only if a signing key is configured)
    internal const string PublicKey = "0024000004800000940000...";
    internal const string PublicKeyToken = "b03f5f7f11d50a3a";
}
```

Since the class is declared as `partial`, you can extend it with your own members in a separate file.

### Accessing ThisAssembly in Other Projects

The `ThisAssembly` class is `internal` by default, so it is only accessible within the project it is generated in. If you need to expose version information to other projects, create a public wrapper or use `[InternalsVisibleTo]`.

### Custom Fields via MSBuild

You can add custom fields to the `ThisAssembly` class using the `AdditionalThisAssemblyFields` item group in your `.csproj` or `Directory.Build.props`:

```xml
<ItemGroup>
  <AdditionalThisAssemblyFields Include="CustomString" String="Hello, World!" />
  <AdditionalThisAssemblyFields Include="CustomBool" Boolean="true" />
  <AdditionalThisAssemblyFields Include="CustomDateTime" Ticks="637505461230000000" />
</ItemGroup>
```

This generates additional members:

```csharp
internal const string CustomString = "Hello, World!";
internal const bool CustomBool = true;
internal static readonly System.DateTime CustomDateTime = new System.DateTime(637505461230000000L, System.DateTimeKind.Utc);
```

## version.json

The package needs a single `version.json` in the root directory, or multiple project based `version.json` files with the configuration of the NBGV. This could be set in root besides the `Directory.Build.props` file with further project details like company info, authors and more ([read more about props files here](project-props-targets.md)).

A simple example could look like this:

```json
{
  "$schema": "https://raw.githubusercontent.com/dotnet/Nerdbank.GitVersioning/master/src/NerdBank.GitVersioning/version.schema.json",
  "version": "1.0",
  "gitCommitIdShortFixedLength": 6,
  "assemblyVersion": {
    "precision": "revision"
  }
}
```

This is the place where you manually set the Major and Minor version numbers. The schema field is optional and should help some editors to use autocompletion for editing the file.

See here to read about all configurable settings: [version.json](https://github.com/dotnet/Nerdbank.GitVersioning/blob/main/doc/versionJson.md)


## DevOps

Use a full clone for your pipeline to avoid shallow clones with:

```yaml
steps:
- checkout: self
  fetchDepth: 0
```

NBGV needs the .git directory to work.

The package has some CI variables by default:

| Build variable                  | property                     | Sample value      |
| ------------------------------- | ---------------------------- | ----------------- |
| GitAssemblyInformationalVersion | AssemblyInformationalVersion | 1.3.1+g15e1898f47 |
| GitBuildVersion                 | BuildVersion                 | 1.3.1.57621       |
| GitBuildVersionSimple           | BuildVersionSimple           | 1.3.1             |

You could even activate more variables with ([cloudbuild doc](https://github.com/dotnet/Nerdbank.GitVersioning/blob/main/doc/cloudbuild.md)):

```yaml
{
  "version": "1.0",
  "cloudBuild": {
    "setVersionVariables": true,
    "setAllVariables": true
  }
}
```


## Coding example

Now you can access the version infos and output them in runtime. 

```csharp
Console.WriteLine($"Data to access with the NBGV class:{Environment.NewLine}" +
  $"AssemblyName: {ThisAssembly.AssemblyName} {Environment.NewLine}" +
  $"AssemblyTitle: {ThisAssembly.AssemblyTitle} {Environment.NewLine}" +
  $"AssemblyVersion: {ThisAssembly.AssemblyVersion} {Environment.NewLine}" +
  $"AssemblyFileVersion: {ThisAssembly.AssemblyFileVersion} {Environment.NewLine}" +
  $"AssemblyInformationalVersion: {ThisAssembly.AssemblyInformationalVersion} {Environment.NewLine}" +
  $"{Environment.NewLine}");

Console.WriteLine($"Finished program. {Environment.NewLine}" +
  $"Company: {FileVersionInfoProcessor.GetFileVersionInfo().CompanyName}, {Environment.NewLine}" +
  $"File Version: {FileVersionInfoProcessor.GetFileVersionInfo().FileVersion}, {Environment.NewLine}" +
  $"Product Version: {FileVersionInfoProcessor.GetFileVersionInfo().ProductVersion}, {Environment.NewLine}Bye bye all!!");
```

[![NBGV Console](/assets/images/articles/DevOps/DevOps_Nbgv_Console.png)](/assets/images/articles/DevOps/DevOps_Nbgv_Console.png)
