---
layout: page
title: DevOps - EF migrations
parent: DevOps
---

# Entity Framework Migrations 

Entity Framework uses Migrations to transport and implement changes of the DB schema from your code. Changes to tables, columns etc. are first built in your code project and then implemented on your DB. 

There are some ways to do that:


## SQL scripts

```batch
dotnet ef migrations script --idempotent
```

This command generates a sql script with all migrations as executable script. The '--idempotent' flag ensures that the script knows the former migrations and looks up the migration's history table in the target DB to ensure that all migrations are done properly. This makes the script more error prone because you don't overwrite or forget a former migration.

See here for more details: [sql-scripts](https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying?tabs=dotnet-core-cli#sql-scripts)



## ef bundles

The EF Bundles are a new approach to apply the migrations and are made to be used as DevOps friendly and production ready way to be used.

```batch
dotnet ef migrations bundle
```

The command generates an exe file called efbundle.exe. This contains the whole migration logic and can be executed alongside the appsettings.json or with a given connection string to access the target DB.


### Example for DevOps


```yaml
trigger:
 branches:
   include:
     - '*'

pool:
  vmImage: 'windows-latest'

variables:
  solution: '**/*.sln'
  buildPlatform: 'Any CPU'
  buildConfiguration: 'Release'
  migrationProjectName: 'YourProjectName'

steps:
- task: NuGetToolInstaller@1
  displayName: 'NuGet Tools Installer'
  inputs:
    checkLatest: true

- task: NuGetCommand@2
  displayName: 'Restore Packages'
  inputs:
    command: 'restore'
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
    Contents: '**/bin/$(buildConfiguration)/**'
    TargetFolder: '$(Build.ArtifactStagingDirectory)/build'

- task: CmdLine@2
  displayName: 'Install dotnet ef tools'
  inputs:
    script: 'dotnet tool update --global dotnet-ef'  
    # update also installs if not installed yet

- task: CmdLine@2
  displayName: 'Generate ef migration bundle'
  inputs:
    script: |
      cd $(migrationProjectName)
      dotnet restore --runtime win-x64 
      dotnet ef migrations bundle --verbose

- task: CopyFiles@2
  displayName: 'Copy migration bundle staging folder'
  inputs:
    SourceFolder: '$(System.DefaultWorkingDirectory)/$(migrationProjectName)'
    Contents: |
      efbundle.exe
      appsettings.json
    TargetFolder: '$(Build.ArtifactStagingDirectory)/migration'

- task: PublishPipelineArtifact@1
  displayName: 'Publish Pipeline Artifacts'
  inputs:
    targetPath: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
    publishLocation: 'pipeline'

```

This complete build pipeline script generates the efbundle.exe and publishes it in the drop/migration folder. The `dotnet restore --runtime win-x64` was added, because without it the bundle command threw an exception, that it would need these.

See here for more details: [bundles](https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying?tabs=dotnet-core-cli#bundles) and [Introducing DevOps-friendly EF Core Migration Bundles](https://devblogs.microsoft.com/dotnet/introducing-devops-friendly-ef-core-migration-bundles/)


## Command-line tools

The CLI tools offer some commands to apply the migrations, too. These are meant to be used for (local) development and testing, but are not recommended to be used for production environment.

```batch
dotnet ef database update
```

See here for more details: [command-line-tools](https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying?tabs=dotnet-core-cli#command-line-tools)


## Apply migrations at runtime or via separate project

Migrations can be triggered on Application startup or via a separeate CLI project. This approach is also not recommended for production environments. 

See here for more details: [apply-migrations-at-runtime](https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying?tabs=dotnet-core-cli#apply-migrations-at-runtime)