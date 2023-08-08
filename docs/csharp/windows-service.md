---
layout: page
title: C# - Windows Service
parent: C#
---

# Windows Service (.NET Framework)

To create a Windows service to be installed and controlled in the services console you can create a console application project (Framework) with some extra classes. Another recommendation is to create an alternative startup for interactive mode, with which you can debug the service without having to install and uninstall it.


## Project Properties

An example for a service with its properties:

[![Project Properties](/assets/images/coding/csharp/windows-service/project-properties.png)](/assets/images/coding/csharp/windows-service/project-properties.png)


## Interactive Mode

This class shows a possible implementation of the interactive mode, with [dependency injection](/docs/csharp/dependency-injection.md), [NerdBank Git Versioning](/docs/DevOps//cicd-versioning-NerdBankGitVersion.md), [Serilog logging](/docs/csharp/logging-in-libraries-nullLogger.md) and stopping a console app with `[Ctrl]+[C]`:

```csharp
internal static class Program
{
    private static bool _keepRunning = true;

    /// <summary>
    /// Host object with dependency injection registration
    /// </summary>
    public static IHost AppHost { get; set; }

    private static void Main()
    {
        var serilogLogger = LogInitializer.CreateLogger(true);
        Log.Logger = serilogLogger;

        Log.Information("{ApplicationName} start", ThisAssembly.AssemblyName);

        AppHost = ConfigureHost();

        ServiceBase[] ServicesToRun;
        ServicesToRun = new ServiceBase[]
        {
            new WindowsServiceEDU() // Your Service class here ...
        };

        if (Environment.UserInteractive)
        {
            try
            {
                RunInteractive(ServicesToRun);
            }
            catch (Exception ex)
            {
                Log.Error(ex, "An error occured while running the service in interactive mode: {Message}", ex.Message);
                throw;
            }
        }
        else
        {
            try
            {
                ServiceBase.Run(ServicesToRun);
            }
            catch (Exception ex)
            {
                Log.Error(ex, "An error occured while running the service: {Message}", ex.Message);
                throw;
            }
        }

        Log.Information("{ApplicationName} stop", ThisAssembly.AssemblyName);
        Log.CloseAndFlush();
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
                // DI registry here ...
            })
            .UseSerilog()
            .Build();
        return host;
    }

    /// <summary>
    /// Runs the application in interactive mode as console application instead of a win serivce.
    /// </summary>
    private static void RunInteractive(ServiceBase[] servicesToRun)
    {
        // Add handler for [Ctrl]+[C] press
        Console.CancelKeyPress += delegate (object sender, ConsoleCancelEventArgs e)
        {
            e.Cancel = true;
            _keepRunning = false;
            Console.WriteLine("Received stop signal, will exit the application ...");
        };

        Console.WriteLine("Services running in interactive mode.");
        Console.WriteLine();

        MethodInfo onStartMethod = typeof(ServiceBase).GetMethod("OnStart", BindingFlags.Instance | BindingFlags.NonPublic);
        foreach (ServiceBase service in servicesToRun)
        {
            Console.WriteLine("Starting {0}...", service.ServiceName);
            onStartMethod.Invoke(service, new object[] { new string[] { } });
            Console.WriteLine("Started");
        }

        Console.WriteLine();
        Console.WriteLine();
        Console.WriteLine("Press [Ctrl]+[C] to exit the application ...");
        while (_keepRunning)
            Thread.Sleep(1000);
        Console.WriteLine();

        MethodInfo onStopMethod = typeof(ServiceBase).GetMethod("OnStop", BindingFlags.Instance | BindingFlags.NonPublic);
        foreach (ServiceBase service in servicesToRun)
        {
            Console.WriteLine("Stopping {0}...", service.ServiceName);
            onStopMethod.Invoke(service, null);
            Console.WriteLine("{0} Stopped", service.ServiceName);
        }

        Console.WriteLine("All services stopped.");
        // Keep the console alive for a second to allow the user to see the message.
        Thread.Sleep(1000);
    }
}
```

## Service component and Installer

[![Service Component](/assets/images/coding/csharp/windows-service/add-service-component.png)](/assets/images/coding/csharp/windows-service/add-service-component.png)

In newer VS editions there are no more Tools from the Toolbox available, so we have to do some tricks for adding the required classes, like the installer and its components. Open the Designer view and then Right Click the background to add the Installer:

[![Service Component](/assets/images/coding/csharp/windows-service/open-designer.png)](/assets/images/coding/csharp/windows-service/open-designer.png)

[![Service Component](/assets/images/coding/csharp/windows-service/add-installer.png)](/assets/images/coding/csharp/windows-service/add-installer.png)

***Note:***
*If you have created this project as console project for the newer .net and as SDK styled project right away, you might get some troubles for adding the serviceInstaller and the serviceProcessInstaller components. The sad thing is that I was not able to find any way to add the serviceInstaller and the serviceProcessInstaller components via the IDE, so I copied them from another project ...* 😒
*So I recommend to setup the project as Framework project and then convert the project to the newer SDK style project file with Targetframework set to net48.*

To configure the installer, you can now set some properties, like the name, the description or if the service will start automatically:

[![Service Component](/assets/images/coding/csharp/windows-service/service-installer-properties.png)](/assets/images/coding/csharp/windows-service/service-installer-properties.png)


### Convert to SDK Style project

I recommend to use the [Try-Convert Tool](/docs/csharp/migrate-sdk-project.md).
Then add the needed Package References and further reduce the csproj entries.


## Install and Uninstall batch scripts

An easy way to install and uninstall the service is to have some batch scripts in the project folder.

***Note:*** Call these from a console with Admin priviledges. Installation and Uninstalltion can only be done by administrators of the machine.


**Install.bat:**

```batch
@ECHO off

REM Set the variables: 
REM  %sourcePath% to be the current directory path
REM  %installUtilPath% for the .NET InstallUtil.exe
REM  %serviceName% for the ServiceName
REM  %serviceUser% for the ServiceUser
SET sourcePath=%cd%
SET installUtilPath="C:\Windows\Microsoft.NET\Framework\v4.0.30319"
SET serviceName=SomeService
SET serviceUser=SomeUser

REM Change to InstalUtils path
C:
CD %installUtilPath%

REM call InstallUtil to install the service
InstallUtil.exe /LogToConsole=true /username=%serviceUser% %sourcePath%\%serviceName%.exe

PAUSE
```

**Uninstall.bat:**

```batch
@ECHO off

REM Set the variables: 
REM  %sourcePath% to be the current directory path
REM  %installUtilPath% for the .NET InstallUtil.exe
REM  %serviceName% for the ServiceName
SET sourcePath=%cd%
SET installUtilPath="C:\Windows\Microsoft.NET\Framework\v4.0.30319"
SET serviceName=SomeService

REM Change to InstalUtils path
C:
CD %installUtilPath%

REM call InstallUtil to install the service
InstallUtil.exe /u /LogToConsole=true %sourcePath%\%serviceName%.exe

PAUSE
```

## service console

[![service console](/assets/images/coding/csharp/windows-service/service-console.png)](/assets/images/coding/csharp/windows-service/service-console.png)