---
layout: page
title: C# - appsettings from PATH EnvVar
parent: C#
---

# Appsettings from PATH EnvVar

Starting or calling a C# application from its location loads the default configuration from the appsettings.json without further problems. But calling the application from anywhere after setting its location in the PATH Environment Variable can cause the ConfigurationBuilder to miss the file and look for it in the directory you currently are located in the console while calling.

Here's a solution for this and an enhancement for not building the config twice if you already built it on the very beginning of your startup for initializing a logger for example.


## PATH Environment Variable

Setting this PATH Env Var enables you to call application from anywhere. Set the variable by GUI or by console command:


### GUI

Open the GUI with the Execute Dialog and type in `rundll32 sysdm.cpl,EditEnvironmentVariables`:

[![Execute GUI](/assets/images/articles/appsettings-PATH-envVar/Execute-GUI.png)](/assets/images/articles/appsettings-PATH-envVar/Execute-GUI.png)

[![EnvVar GUI](/assets/images/articles/appsettings-PATH-envVar/EnvVar-GUI.png)](/assets/images/articles/appsettings-PATH-envVar/EnvVar-GUI.png)


### Console 

I have another article covering this at [win-powershell-set-env-var](/docs/other/win-powershell-set-env-var.md), but in short you could use this PowerShell command:

```shell
[System.Environment]::SetEnvironmentVariable('PATH','Your_Path',[System.EnvironmentVariableTarget]::Machine)
```


## ConfigurationBuilder Enhancement

If you use Serilog or some logging framework, that gets configs from the appsettings.json, you might build your appConfig at the very start of your application. Your code might look something like this:

```csharp
public static IConfigurationBuilder GetAppConfigBuilder()
{
    var envVarName = GetEnvVarName();
    var appConfigBuilder = new ConfigurationBuilder()
        .SetBasePath(AppContext.BaseDirectory)
        .AddJsonFile("appsettings.json")
        .AddJsonFile($"appsettings.{Environment.GetEnvironmentVariable(envVarName)}.json", optional: true, reloadOnChange: true)
        .AddEnvironmentVariables();

    return appConfigBuilder;
}
```

But if you use the `Host.CreateDefaultBuilder()` further down, you might build this config again there. But more problematic is that the DefaultBuilder does not set the BasePath with `.SetBasePath(AppContext.BaseDirectory)`. So it does not find anything if you call the application with the prior set PATH envVar.

The solution for both these issues is to pass the already built configuration object to the DefaultBuilder:

```csharp
var host = Host.CreateDefaultBuilder()
    .ConfigureAppConfiguration(builder =>
    {
        builder.Sources.Clear();
        builder.AddConfiguration(appConfig); // <-- pass the config with set BasePath
    })
    .ConfigureServices((context, services) =>
    {
        // ...
    })
    .UseSerilog()
    .Build();
```

