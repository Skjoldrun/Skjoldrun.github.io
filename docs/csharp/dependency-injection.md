---
layout: page
title: C# - Dependency Injection
parent: C#
---

# Dependency Injection

Dependency Injection is an implementation of the Dependency Inversion Principle, wich stands for the D in [SOLID](https://en.wikipedia.org/wiki/SOLID).

With Dependency Inversion classes "wish" for their dependencies and the using caller has to deliver these dependencies on instantiation.

A class built with this principle can use Dependency Injection (DI) with its constructor (or even with its Properties). The class then expects interfaced parameters of its dependencies and maps them to its fields. These hold the later given implementations of the dependencies. This enables a loose coupling of the components and has the advantage of being less dependent on explicit implementations. For using Dependency Injection in your application code, you need a system to know which implementation belongs to which interface to be implemented when instanciating a class. Such a registratration system then also instantiates the classes for you and injects the propper dependencies into it. Such systems are often refered to as IoC (Inversion of Control) container, like [Autofac](https://autofac.org/) or [Ninject](https://www.ninject.org/). But you can use Microsofts Extensions.DependencyInjection system as well!

Let's see an example for that:

```csharp
namespace DependencyInjectionExample
{
    internal class Program
    {
        public static IHost AppHost { get; set; }

        private static void Main(string[] args)
        {
            // Create serilog logger with custom Initializer class for configuration
            var serilogLogger = new LoggerConfiguration()
                .WriteTo.Console()
                .CreateLogger();

            // Set this logger als the static logger
            Log.Logger = serilogLogger;

            Log.Information("Application start");

            AppHost = ConfigureHost();

            // Resolve an instance of a registered service and start it
            var service = AppHost.Services.GetService<IProjectExampleClass>();
            service.ExampleMethod();

            // Resolve an instance of a registered service and start it
            var libService = AppHost.Services.GetService<ILibExampleClass>();
            libService.LibExampleMethod();

            // Create a parameter object, this could be an object array
            var parameter = true;
            // Create an instance of a registered service with onjected logger and inject the parameters (order must match the ctor order)
            var libServiceParam = ActivatorUtilities.CreateInstance<LibExampleWithParam>(AppHost.Services, parameter);
            libServiceParam.GetSomeStringAnswer();

            Log.Information("Application stop");
            Log.CloseAndFlush();

            System.Console.WriteLine("Enter any key to end ...");
            System.Console.ReadKey();
        }

        /// <summary>
        /// Configures the host with registering the interfaces and class types.
        /// Sets the Serilog logger as logging provider for typed ILogger injections.
        /// </summary>
        /// <returns>configured host to access its services</returns>
        private static IHost ConfigureHost()
        {
            var host = Host.CreateDefaultBuilder()
                .ConfigureServices((context, services) =>
                {
                    // Define DI here ...
                    // AddTransient gives a new instance everytime it's called
                    services.AddTransient<IProjectExampleClass, ProjectExampleClass>();

                    // Add library DI registration extension methods here ...
                    services.AddDiExampleLibServices();
                })
                .UseSerilog()  // Serilog.Extensions.Hosting -> Use Serilog Logger instead of MS.Extensions.Logger at all ILogger injections
                .Build();
            return host;
        }
    }
}
```

The example is about an console application with Serilog Logging, `Microsoft.Extensions.Logging`, `Microsoft.Extensions.Hosting` and `Microsoft.Extensions.DependencyInjection`. The first step is to configure the Serilog logger, the second step is to configure the application host to use the Serilog logger and to register all needed services with their interfaces. This is the place where the mapping and the lifetime scope for instances of classes is made. With `services.AddTransient<IProjectExampleClass, ProjectExampleClass>();` a new instance of type `ProjectExampleClass` is spined up every time the system is asked for something of type `IProjectExampleClass`. You can register for singletons with `services.AddSingleton`, too. That's the place where wishes of dependencies come true! 😉
As third step, we then call the `AppHost.Services.GetService<IProjectExampleClass>()` method to get a new instance of the demanded type and start calling one of its methods.

If you use DI in your application you stop instantiating classes with the new keyword but let the DI system instantiate them with propper resolved dependencies for you. 

One of the called example methods is from the app internally and one is from a class library:

```csharp
using Microsoft.Extensions.Logging;

namespace DependencyInjectionExample.ExampleClasses
{
    public class ProjectExampleClass : IProjectExampleClass
    {
        private readonly ILogger<ProjectExampleClass> _logger;

        public ProjectExampleClass(ILogger<ProjectExampleClass> logger)
        {
            _logger = logger;
        }

        public void ExampleMethod()
        {
            _logger.LogInformation("this method does something interessting ...");
        }
    }
}
```

```csharp
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Logging.Abstractions;

namespace DIExampleLib
{
    public class LibExampleClass : ILibExampleClass
    {
        private readonly ILogger<LibExampleClass> _logger;

        public LibExampleClass(ILogger<LibExampleClass> logger = null)
        {
            _logger = logger ?? NullLogger<LibExampleClass>.Instance;
        }

        public void LibExampleMethod()
        {
            _logger.LogInformation("Lib does something very interessting!");
        }
    }
}
```

The library class only needs the `Microsoft.Extensions.Logging` and `Microsoft.Extensions.Logging.Abstractions` usings.

To register this library class, it is common to write some extension method for the DI registration, to be called in the application "root of configuration". A extension method could look something like this:

```csharp
using Microsoft.Extensions.DependencyInjection;

namespace DIExampleLib.Utilities
{
    /// <summary>
    /// Class for extending the dependency injection registration of the caller with the library registrations.
    /// </summary>
    public static class DiExampleServiceExtension
    {
        /// <summary>
        /// Registers the class library interfaces and implementations as extension method for the calling application.
        /// </summary>
        public static IServiceCollection AddDiExampleLibServices(this IServiceCollection services)
        {
            services.AddTransient<ILibExampleClass, LibExampleClass>();

            return services;
        }
    }
}
```


## Create an instance with constructor parameters

If you want to create instances of your classes, which should receive situation dependent parameters via constructor, you can inject them with the `ActivatorUtilities.CreateInstance<YourClass>(AppHost.Services, parameters)` call:

The class constructor expects to get some parameters, for example a bool flag to decide certain execution steps:

```csharp
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Logging.Abstractions;

namespace DIExampleLib.LibExampleClasses
{
    public class LibExampleWithParam : ILibExampleWithParam
    {
        private readonly ILogger<LibExampleWithParam> _logger;
        private readonly bool _someBoolParam;

        public LibExampleWithParam(ILogger<LibExampleWithParam> logger = null, bool someBoolParam = false)
        {
            _logger = logger ?? NullLogger<LibExampleWithParam>.Instance;
            _someBoolParam = someBoolParam;
        }

        public void GetSomeStringAnswer()
        {
            if (_someBoolParam == true)
                _logger.LogInformation("Param was set with constructor.");
            else
                _logger.LogInformation("Param might be set with constructor or remained the default value.");
        }
    }
}
```

This parameter can be injected with the call of `ActivatorUtilities.CreateInstance<YourClass>(AppHost.Services, parameters)` in the Program.cs class. If you wnat to inject multiple parameters, you have to build an object array with ordered parameters to be injected one after the other.

![console out](/assets/images/coding/csharp/dependency-injection/console-output.png)

If you have to instantiate multiple new objects from a class to be stored in another class, you can then get these new instances with the `ActivatorUtilities.CreateInstance<YourClass>(AppHost.Services)` method instead of calling `new YourClass( ??? )` and have to fiddle around with the dependencies to fill in the constructor here. But this has a little drawback: you have to drag the dependency on the `AppHost.Services` as `IServiceProvider` all the way down to the place you new up those instances. This can lead to your library becoming dependent of using Dependency Injection for the consuming application... oh the irony!
