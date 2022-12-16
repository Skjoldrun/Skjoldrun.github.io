---
layout: page
title: C# - appsettings and environments
parent: C#
---

# Appsettings and Environments

In .NET core and newer the old app.config as XML file gets replaced by the appsettings.`<environment>`.json file to store settings, like logger configurations, connectionstrings and more. The `<environment>` part can determine for which scenario the file gets loaded. You could write "Production", or "development" files to load different settings for the application.

## Setup

### Files

You can add this file via VS context menu and choose the JavaScript-JSON-Configuration file. Change the properties of the file to "allways copy" or to "copy if newer" to place it in the build directory. You can also change this in the project file:

[![appsettings allway copy](/assets/images/coding/csharp/appsettings-environment/project-settings.png)](/assets/images/coding/csharp/appsettings-environment/project-settings.png)

If you have a different appsettings file for debug purposes you can control if it will be copied to the output directory with the project settings, too. 

```xml
<Choose>
<When Condition="'$(Configuration)' == 'Debug'">
    <ItemGroup>
        <None Include="appsettings.json" 
        CopyToOutputDirectory="Always" 
        CopyToPublishDirectory="Always" />
        <None Include="appsettings.Development.json" 
        CopyToOutputDirectory="Always" 
        CopyToPublishDirectory="Always" />
    </ItemGroup>
</When>
<When Condition="'$(Configuration)' == 'Release'">
    <ItemGroup>
        <None Include="appsettings.json" 
        CopyToOutputDirectory="Always" 
        CopyToPublishDirectory="Always" />
        <None Include="appsettings.Development.json" 
        CopyToOutputDirectory="Never" 
        CopyToPublishDirectory="Never" />
    </ItemGroup>
</When>
</Choose>
```

The conditions control the copy process via debug or release settings. 

[![file dependency](/assets/images/coding/csharp/appsettings-environment/file-dependency.png)](/assets/images/coding/csharp/appsettings-environment/file-dependency.png)

You can set a dependency of the two files to show them in a hierarchy in the project structure:

```xml
<ItemGroup>
    <None Update="appsettings.Development.json">
        <DependentUpon>appsettings.json</DependentUpon>
    </None>
</ItemGroup>
```

[![file dependency](/assets/images/coding/csharp/appsettings-environment/file-dependency-set.png)](/assets/images/coding/csharp/appsettings-environment/file-dependency-set.png)


### Environment Variables

Create the file Properties\LaunchSettings.json and write the following entries to activate the Development environment variable:

```json
{
  "profiles": {
    "YOUR_PROJECT_NAME": {
      "commandName": "Project",
      "environmentVariables": {
        "DOTNET_ENVIRONMENT": "Development"
      }
    }
  }
}
```
You can also set this up via UI in VS:

[![launchsettings](/assets/images/coding/csharp/appsettings-environment/launchsettings-gui.png)](/assets/images/coding/csharp/appsettings-environment/launchsettings-gui.png)


### Configuration and host builder

Create the configuration variable from the appsettings file[s]: 

```csharp
// Build a configuration from appsettings.json files and store them in the var
var configuration = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("appsettings.json")
    .AddJsonFile($"appsettings.{Environment.GetEnvironmentVariable("DOTNET_ENVIRONMENT")}.json", true)
    .Build();
```

This examples also uses the environment variable `DOTNET_ENVIRONMENT` which should be set for each mode you are working with. If you switch to release this does not throw an exception for a missing file, because the call for `.AddJsonFile($"appsettings.{Environment.GetEnvironmentVariable("DOTNET_ENVIRONMENT")}.json", true)` uses true as flag for optional.


### use with serilog logging

If you use serilog you can use the different file for debug and release as well. This also enables logging even before setting up the Host Builder for further setups.

```csharp
// Create logger with above configuration
Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(configuration)
    .Enrich.WithMachineName()
    .Enrich.WithProcessName()
    .Enrich.WithThreadId()
    .Enrich.WithAssemblyVersion()
    .WriteTo.MSSqlServer(
        //connectionString: ConnectionStringBuilder.GetFromConfigOrDefault(configuration),
        connectionString: configuration.GetConnectionString("LogConnectionString"),
        new MSSqlServerSinkOptions
        {
            SchemaName = SchemaName,
            TableName = TableName,
            AutoCreateSqlTable = true
        },
        columnOptions: BuildColumnOptions())
    .CreateLogger();
```


### Host builder with serilog

Now you only have to build the host with the serilog integration and register your DI services:

```csharp
// Build host with Dependency Injection and Serilog as logger
var host = Host.CreateDefaultBuilder()
    .ConfigureServices((context, services) =>
    {
        // define DI here ...
        services.AddTransient<IYourService, YourService>();
    })
    .UseSerilog() // <- Serilog
    .Build();
```
