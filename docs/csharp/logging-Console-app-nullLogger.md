---
layout: page
title: C# - Logging, DI, NullLogger
parent: C#
---

# Logging in ConsoleApps with Dependency Injection & NullLogger

In the article [Logging in libraries, NullLogger](csharp-logging-in-libraries-nullLogger.md) I have described the concepts of logging with a manual injection of the logger instance by the Microsoft.Extensions.Logging LoggerFactory.

But how do you log with Serilog without explicitly calling the factory and injecting the logger by yourself?

I have come to the following pattern to configure my logging and Dependency Injection in my applications:


## Configuring the Host with DI and Serilog

***Note:** This is also usable in a .NET Framework Console application!*

This may be familiar to you if you have built ASP.NET, Webservice or API applications. In the beginning, you configure your host with the `Host.CreateDefaultBuilder()` Method from `Microsoft.Extensions.Hosting`. This already does a lot of things for you to build your default host, but we want to configure a little bit more.

If we want to use a logging framework, like Serilog, it must be configured first:


**Serilog configuration example:**

```csharp
public static class LogInitializer
{
    public static ILogger CreateLogger()
    {
        return new LoggerConfiguration()
            .ReadFrom.AppSettings()
            .Enrich.FromLogContext()
            .CreateLogger();
    }
}
```

This is an example to create a minimalistic Serilog logger with the `.ReadFrom.AppSettings()` from `Serilog.Settings.AppSetting` to read settings from the app.config file (there's a Package for using appsettings.json as well):

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
	<startup>
		<supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.7" />
	</startup>
	<appSettings>
		<add key="serilog:minimum-level" value="Debug" />
		<add key="serilog:using:Debug" value="Serilog.Sinks.Debug" />
		<add key="serilog:write-to:Debug" />
		<add key="serilog:using:Console" value="Serilog.Sinks.Console" />
		<add key="serilog:write-to:Console" />
	</appSettings>
</configuration>
```

Serilog gets configured to write to Console and to Debug Out with this. The `LogInitializer.CreateLogger()` Method can now be called in the Main Method:

```csharp
private static void Main(string[] args)
{
    // Create serilog logger with custom Initializer class for configuration
    Log.Logger = LogInitializer.CreateLogger();

    Log.Information("Application start");

    // Configure host with Dependency Injection registration and Serilog as logger
    IHost host = ConfigureHost();

    // Get service and start it by calling its methods
    var service = ActivatorUtilities.GetServiceOrCreateInstance<ISomeClass>(host.Services);
    service.SomeMethod();

    Log.Information("Application stop");
    Log.CloseAndFlush();

    Console.ReadKey();
}
```

To use the Serilog logger as default logger, we must configure it for the host. This happens in the `ConfigureHost()` method with `.UseSerilog()` from `Serilog.Extensions.Logging`:

```csharp
/// <summary>
/// Configures the host with registering the interfaces and class types.
/// Sets the Serilog logger as logging provider for typed ILogger innjections.
/// </summary>
/// <returns>configured host to access its services</returns>
private static IHost ConfigureHost()
{
    var host = Host.CreateDefaultBuilder()
        .ConfigureServices((context, services) =>
        {
            // define DI here ...
            // AddTransient gives a new instance everytime it's called
            services.AddTransient<ISomeClass, SomeClass>();
            services.AddTransient<ISomeOtherClass, SomeOtherClass>();
        })
        .UseSerilog()  // Serilog.Extensions.Hosting -> Use Serilog Logger instead of MS.Extensions.Logger at all ILogger injections
        .Build();
    return host;
}
```

The upper code also shows the registration of interfaces and their class types for dependency injection, where the `ISomeClass` is registered as `SomeClass` to be resolved.
You can use `.AddTransient<>()` to allways create a new instance, or `.AddSingleton<>()` for a singleton instance to be used all over the application.


## Dependency Injection by constructor

With Dependency Injection by constructor you define the class to ask for the wanted dependency with an interface. In this case we want an `ILogger` interface of the class type we want to log with:

```csharp
public class SomeClass : ISomeClass
{
    private readonly ILogger _logger;

    public SomeClass(ILogger<SomeClass> logger = null)
    {
        _logger = logger ?? NullLogger<SomeClass>.Instance;
    }

    public void SomeMethod()
    {
        _logger.LogInformation("Method was called.");
        // Do something ...
    }
}
```

This class can be located in a logging framework agnostic library without Serilog as dependency and still is able to get the `ILogger` instance and write out logs:

[![console logging output](/assets/images/articles/logging-Console-app-nullLogger/console-logging.png)](/assets/images/articles/logging-Console-app-nullLogger/console-logging.png)


