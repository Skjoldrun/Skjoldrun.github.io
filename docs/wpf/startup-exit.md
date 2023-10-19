---
layout: page
title: WPF - startup and exit
parent: WPF
---

# startup and exit

The default setup has a startupUri defined in the App.xaml and closes the application on closing the main window. If you want to open other Views and exit explicitly, you have to change some values:

[![App.xaml](/assets/images/articles/startup-exit/app.xaml.png)](/assets/images/articles/startup-exit/app.xaml.png)


## Startup and exit methods

The startupUri can be changed to another View of your choice, but can also be extended or changed to a Startup method: `Startup="Application_Startup"`.

This also applies to the exit of the app, which can be extended with a custom method, too: `Exit="Application_Exit"`.

[![App.xaml](/assets/images/articles/startup-exit/startup-exit-methods.png)](/assets/images/articles/startup-exit/startup-exit-methods.png)

This is a way to init a logger on startup and flush it on exiting the app.
But you can also use the `OnStartup` and `OnExit` mehtods to prepare you application or do same final work on exiting it. Heres an example which shows the order of execution if all four methods are on the application:

[![method-order](/assets/images/articles/startup-exit/method-order.png)](/assets/images/articles/startup-exit/method-order.png)


## explicit shutdown

If you don't want to close the program on closing the last or the main window, you have to define an explicit shutdown. Add `ShutdownMode="OnExplicitShutdown"` in the App.xaml to achieve this:

[![explicit-shutdown](/assets/images/articles/startup-exit/explicit-shutdown.png)](/assets/images/articles/startup-exit/explicit-shutdown.png)

This can be used if you have implemented a systemtray icon to control your app with a context menu, like I have done in my projects [Winsomnia](https://github.com/Skjoldrun/Winsomnia) or [StandUpMate](https://github.com/Skjoldrun/StandUpMate):

[![system tray menu](/assets/images/articles/startup-exit/system-tray-menu.png)](/assets/images/articles/startup-exit/system-tray-menu.png)
