---
layout: page
title: WPF - startup, exit, double start check
parent: WPF
---


# Controlled startup in a WPF application

WPF has some different alternatives to start the application. A clean way to do it controlled is the following:

In WPF you can define a startupUri in the `App.xaml` file and register startup and exit methods there. I will describe this later, but can't recommend this approach any more, because it parallelizes the startup of the View to the startup Methods and therefore can create race conditions and chaos.

The better approach is to simply leave the `App.xaml` file as it is and utilize the `OnStartup` and `OnExit` methods in the App.xaml.cs file.


## OnStartup

The `protected override [async] void OnStartup(StartupEventArgs e)` method gets raised directly on startup and is the place where you can init the logger, host, DI and additional app services. Here you can then also init and open the main View and open it as window:

```csharp
protected override async void OnStartup(StartupEventArgs e)
{
    Log.Logger = LogInitializer.CreateLogger(selfLog: true);
    Log.Information("{ApplicationName} start", ThisAssembly.AssemblyName);

    var splash = new SplashScreenView();
    splash.Show();
    await Dispatcher.Yield();

    try
    {
        if (CheckIfDoubleStarted())
        {
            CloseApplication();
            return;
        }

        AppDomain.CurrentDomain.UnhandledException += new UnhandledExceptionEventHandler(AppUnhandledException);

        if (splash.DataContext is SplashScreenViewModel vm)
            vm.StatusText = "Starting ...";
        await InitializeApplicationAsync();

        // ...

        MainWindow = new MainWindowView();
        MainWindow.Closing += new CancelEventHandler(OnClosingWindow);
        var mainVM = new MainWindowViewModel(MainWindow);
        MainWindow.DataContext = mainVM;
        splash.Close();

        base.OnStartup(e);
        MainWindow.ShowDialog();
    }
    catch (Exception ex)
    {
        MessageBox.Show(ex.Message, "Startup error", MessageBoxButton.OK, MessageBoxImage.Error);
        splash.Close();
        Log.Error(ex, ex.Message);
        CloseApplication();
        return;
    }
}
```

As you can see, the DataContext and the `OnClosingWindow` are registered manually and therefore you have full control about the order of things being loaded and started. In this example you even have an optional splash screen and an async init call for keeping the main UI thread non blocked.

The async `InitializeApplicationAsync()` method could be like:
```csharp
/// <summary>
/// Initialize application services and properties in the background.
/// </summary>
private async Task InitializeApplicationAsync()
{
    await Task.Run(() =>
    {
        AppHost = ConfigureHost();
        // init stuff here ...
    });
}
```


## Check if double started with Mutex

Checking if the application was started before can prevent double usage of connections or other unwanted behavior. Using a Mutex for this is fast and is controlled by the OS, so that the mutex will be cleaned even after the app would crash.

```csharp
/// <summary>
/// Checks if the application was started before and closes it to prevent two or multiple instances.
/// </summary>
private static bool CheckIfDoubleStarted()
{
    bool isDoubleStarted = false;
    try
    {
        // Local mutex with unique app id to compare
        string mutexName = $@"Local\{ThisAssembly.AssemblyName}_MutexID";
        _singleInstanceMutex = new Mutex(initiallyOwned: true, mutexName, out bool isFirstInstance);

        if (isFirstInstance == false)
        {
            MessageBox.Show($"{ThisAssembly.AssemblyName} läuft bereits, aber kann nur einmal gestartet werden.",
                $"{ThisAssembly.AssemblyName}", MessageBoxButton.OK, MessageBoxImage.Error);
            Log.Information("Application {AssemblyName} was double started", ThisAssembly.AssemblyName);

            isDoubleStarted = true;
        }
    }
    catch (UnauthorizedAccessException ex)
    {
        // Mutex exists but ACL does not allow access
        Log.Error(ex, "Local mutex exists but access was denied. Exclusive start not guaranteed.");
    }
    catch (SecurityException ex)
    {
        // Security policy prevents mutex creation
        Log.Error(ex, "Security policy prevents Local mutex creation. Exclusive start not guaranteed.");
    }

    return isDoubleStarted;
}
```

This example just shows a `MessageBox` to the user adn closes the application gracefully.


## Closing the application

```csharp
/// <summary>
/// Logs the program end and flushes remaining log entries to the sinks.
/// </summary>
private void OnClosingWindow(object sender, CancelEventArgs e)
{
    CloseApplication();
}

/// <summary>
/// Logs application ending, flushes logger and closes the application per shutdown.
/// </summary>
public static void CloseApplication()
{
    CheckAndCloseOpcUaClient();

    if (ProdEventWriter != null)
    {
        ProdEventWriter.Insert(
            machineId: (uint)MachineSettingsConfig.MachineSettings.MischerNummer,
            eventId: ProdEvent.evt_ereignis_stop_programm,
            timeStamp: DateTime.Now,
            comment: ProdEvent.evt_ereignis_stop_programm_str,
            string1: AppVersionName);

        SpinWait.SpinUntil(() => ProdEventWriter.BufferedElementsCount() == 0,
                new TimeSpan(0, 0, MachineSettingsConfig.MachineSettings.WaitForOfflineWriterInSeconds));
    }

    Log.Information("{ApplicationName} stop", ThisAssembly.AssemblyName);
    Log.CloseAndFlush();
    Current.Shutdown();
}
```


# startupUri, startup and exit *(Not recommended!)*

The worse alternative way uses the startupUri defined in the `App.xaml` and closes the application on closing the main window. If you want to open other Views and exit explicitly, you would have to change some values:

[![App.xaml](/assets/images/articles/startup-exit/app.xaml.png)](/assets/images/articles/startup-exit/app.xaml.png)


## Startup and exit methods

The startupUri can be changed to another View of your choice, but can also be extended or changed to a Startup method: `Startup="Application_Startup"`.

This also applies to the exit of the app, which can be extended with a custom method, too: `Exit="Application_Exit"`.

[![App.xaml](/assets/images/articles/startup-exit/startup-exit-methods.png)](/assets/images/articles/startup-exit/startup-exit-methods.png)

This is a way to init a logger on startup and flush it on exiting the app.
But you can also use the `OnStartup` and `OnExit` methods to prepare you application or do same final work on exiting it. Heres an example which shows the ideal order of execution if all four methods are on the application *(But be aware that this is not guaranteed and the opening of the view can be parallel to the OnStartup event!)*:

[![method-order](/assets/images/articles/startup-exit/method-order.png)](/assets/images/articles/startup-exit/method-order.png)


## explicit shutdown

If you don't want to close the program on closing the last or the main window, you have to define an explicit shutdown. Add `ShutdownMode="OnExplicitShutdown"` in the App.xaml to achieve this:

[![explicit-shutdown](/assets/images/articles/startup-exit/explicit-shutdown.png)](/assets/images/articles/startup-exit/explicit-shutdown.png)

This can be used if you have implemented a systemtray icon to control your app with a context menu, like I have done in my projects [Winsomnia](https://github.com/Skjoldrun/Winsomnia) or [StandUpMate](https://github.com/Skjoldrun/StandUpMate):

[![system tray menu](/assets/images/articles/startup-exit/system-tray-menu.png)](/assets/images/articles/startup-exit/system-tray-menu.png)
