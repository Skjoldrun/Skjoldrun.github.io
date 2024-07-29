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

For example, if the version specified at HEAD is 1.1 and the longest path in git history from HEAD to where the version file was changed to 1.1 includes 15 commits, then the git height is "15" and teh Version would look something like this: `1.1.15`.


## Additional Information

A very nice addition is that the tool injects a class called `ThisAssembly` which then holds the following infos to be accessed as internal const strings:

```csharp
internal sealed partial class ThisAssembly {
    internal const string AssemblyVersion = "1.0";
    internal const string AssemblyFileVersion = "1.0.24.15136";
    internal const string AssemblyInformationalVersion = "1.0.24-alpha+g9a7eb6c819";
    internal const string AssemblyName = "Microsoft.VisualStudio.Validation";
    internal const string PublicKey = @"0024000004800000940000...reallylongkey..2342394234982734928";
    internal const string PublicKeyToken = "b03f5f7f11d50a3a";
    internal const string AssemblyTitle = "Microsoft.VisualStudio.Validation";
    internal const string AssemblyConfiguration = "Debug";
    internal const string RootNamespace = "Microsoft";
}
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


# Pipeline Example using library vars

Here's an example of a pipeline that I use in my projects combined with the [library variables](/docs/DevOps/cicd-library-vars.md):

```yaml
trigger:
 branches:
   include:
     - 'master'

pool:
  vmImage: 'windows-latest'

variables:
  - group: YOUR_GROUP_NAME
  - name: SolutionName
    value: YOUR_SOLUTION_NAME
  - name: Vars_GroupId
    value: 'YOUR_GROUP_ID'
  - name: NBGV_Version
    value: 'NBGV_$(SolutionName)'
  - name: solution
    value: '**/*.sln'
  - name: buildPlatform
    value: 'Any CPU'
  - name: buildConfiguration
    value: 'Release'

steps:
- checkout: self
  submodules: true
  persistCredentials: true
  fetchDepth: 0

- task: NuGetToolInstaller@1
  displayName: 'Install NuGet Tools'
  inputs:
    checkLatest: true

- task: NuGetCommand@2
  displayName: 'Restore NuGet Packages'
  inputs:
    restoreSolution: '$(solution)'

- task: VSBuild@1
  displayName: 'Build Solution'
  inputs:
    solution: '$(solution)'
    platform: '$(buildPlatform)'
    configuration: '$(buildConfiguration)'

- task: CopyFiles@2
  displayName: 'Copy build artifacts to staging folder'
  inputs:
    SourceFolder: '$(System.DefaultWorkingDirectory)'
    Contents: '$(SolutionName)/bin/$(buildConfiguration)/**'
    TargetFolder: '$(Build.ArtifactStagingDirectory)'

- task: PublishPipelineArtifact@1
  displayName: 'Publish Pipeline Artifacts'
  inputs:
    targetPath: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
    publishLocation: 'pipeline'

- powershell: |
    dotnet tool install --tool-path . nbgv
  displayName: Install NBGV CLI Tools

# Service-TFSBuild user needs the azure CLI with DevOps extension. Install with 'az extension add --name azure-devops'
- powershell: |
    Write-Host "Update library variable '$(NBGV_Version)' with NBGV var GitBuildVersion $(GitBuildVersion) ..."
    az pipelines variable-group variable update --group-id $(Vars_GroupId) --name $(NBGV_Version) --value "$(GitBuildVersion)"
  displayName: Update library NBGV Version Variable
  env:
    AZURE_DEVOPS_EXT_PAT: $(System.AccessToken)
```