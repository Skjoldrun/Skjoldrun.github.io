---
layout: page
title: C# - Dependency Injection in WPF
parent: C#
---

# Dependency Injection in WPF

I wrote a base article to Dependency Injection [here](/docs/csharp/dependency-injection.md). This article takes the topic a little bit further and shows how to add DI into a WPF project.

First of all we delete the default start of the MainWindow from the `App.xaml` file:

```xml
<Application x:Class="OutlookInteroptTester.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:OutlookInteroptTester">
    <Application.Resources>
    </Application.Resources>
</Application>
```

After that, we add the NuGet packages `Microsoft.Extensions.Hosting` and `Microsoft.Extensions.DependencyInjection`. Then we add the default startup logic for the `App.xaml.cs` file:

```csharp
public partial class App : Application
{
    public static IHost? AppHost { get; private set; }

    public App()
    {
        AppHost = Host.CreateDefaultBuilder()
            .ConfigureServices((hostContext, services) =>
            {
                services.AddSingleton<MainWindow>();
                services.AddFactory<ChildView>();
            })
            .Build();
    }

    protected override async void OnStartup(StartupEventArgs e)
    {
        await AppHost!.StartAsync();

        var startupForm = AppHost.Services.GetRequiredService<MainWindow>();
        startupForm.Show();

        base.OnStartup(e);
    }

    protected override async void OnExit(ExitEventArgs e)
    {
        await AppHost!.StopAsync();
        base.OnExit(e);
    }
}
```

This code lets you start with the MainWindow View and exits the application on closing it as last opened window.


# Abstract Factory for opening further Views

If we want to open Chield Views as seperate Windows, we could use a abstract factory to achieve that. Whithout this we would only be able to open one window from any Chield View.

We add the folder "StartupHelpers" with a class called AbstractFactory in it:

```csharp
using System;

namespace OutlookInteroptTester.StartupHelpers;

public class AbstractFactory<T> : IAbstractFactory<T>
{
    private readonly Func<T> _factory;

    public AbstractFactory(Func<T> factory)
    {
        _factory = factory;
    }

    public T Create()
    {
        return _factory();
    }
}
```

THis class gets an interface added and then we extend the Dependency Injection with a ServiceExtension class:

```csharp
using Microsoft.Extensions.DependencyInjection;
using System;

namespace OutlookInteroptTester.StartupHelpers;

public static class ServiceExtensions
{
    public static void AddFactory<TForm>(this IServiceCollection services)
        where TForm : class
    {
        services.AddTransient<TForm>();
        services.AddSingleton<Func<TForm>>(x => () => x.GetService<TForm>()!);
        services.AddSingleton<IAbstractFactory<TForm>, AbstractFactory<TForm>>();
    }
}
```

Through the extensions we can get any Form or Window class and generate the factory for it as a singleton with the delegated create method.

The first registration is the implementation of the type to register. Its the View in this case, but it could also be the typical interface and implementation combination, like `services.AddTransient<TInterface, TImplementation>();` (*This would need another method signature*).
The second registration with the code `Func<TForm>>(x => () => x.GetService<TForm>()!)` is a delegate to be called and executed by the factory to get the registered instance of the generic type. 
The third registration is the abstract factory itself, to be passed in a class which needs to instanciate the dependency, the view in this case.

Man, this is somehow complex ... 🤯 


## Using the factory to open a new view

The Mainwindow is now dependent on a factory to open multiple parallel ClientViews:

```csharp
public partial class MainWindow : Window
{
    private readonly IAbstractFactory<ChildView> _chieldViewFactory;

    public MainWindow(IAbstractFactory<ChildView> chieldViewFactory)
    {
        InitializeComponent();
        _chieldViewFactory = chieldViewFactory;
    }

    private void OpenChildView_Click(object sender, RoutedEventArgs e)
    {
        _chieldViewFactory.Create().Show();
    }
}
```

On the button click event the factory creates the instance and shows it.

