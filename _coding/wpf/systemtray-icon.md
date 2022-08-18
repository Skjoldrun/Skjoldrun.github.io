---
layout: page
title: WPF - systemtray icon
---

# Systemtray Icon

WPF has no built in Systemtray or Notify Icon like WinForm does. If you want to have this for your app, you could use [wpf-notifyicon](https://github.com/hardcodet/wpf-notifyicon) from hardcodet.

I use this from my [Winsomnia](https://github.com/Skjoldrun/Winsomnia) and [StandUpMate](https://github.com/Skjoldrun/StandUpMate) projects, too.

![StandUpMate Systemtray Icon](/assets/images/coding/wpf/startup-exit/system-tray-menu.png)


## Setup

Create a ressource dictionary in the App.xaml file:

![StandUpMate Systemtray Icon](/assets/images/coding/wpf/systemtray-icon/ressourcedic-app.xaml.png)

```xml
<Application x:Class="StandUpMate.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:StandUpMate"
             ShutdownMode="OnExplicitShutdown">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="Resource/NotifyIconResources.xaml" />
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

Change the ShutdownMode to `ShutdownMode="OnExplicitShutdown"` to control the app via context menu of your systemtray icon. 


**Example for the Resource/NotifyIconResources.xaml**

```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                    xmlns:tb="http://www.hardcodet.net/taskbar"
                    xmlns:vm="clr-namespace:StandUpMate.ViewModel">

    <ContextMenu x:Shared="false" x:Key="SysTrayMenu">
        <MenuItem Header="Start timer" Command="{Binding StartTimerCommand}" />
        <MenuItem Header="Stop timer" Command="{Binding StopTimerCommand}" />
        <MenuItem Header="Show remaining time" Command="{Binding ShowRemainingTimeCommand}" />
        <Separator />
        <MenuItem Header="Settings" Command="{Binding ShowSettingsWindowCommand}" />
        <MenuItem Header="Close" Command="{Binding ExitApplicationCommand}" />
    </ContextMenu>

    <tb:TaskbarIcon x:Key="NotifyIcon"
                    IconSource="/Resource/clock.ico"
                    ToolTipText="StandUpMate"
                    LeftClickCommand="{Binding LeftClickCommand}"
                    DoubleClickCommand="{Binding ShowSettingsWindowCommand}"
                    ContextMenu="{StaticResource SysTrayMenu}">

        <tb:TaskbarIcon.DataContext>
            <vm:NotifyIconViewModel />
        </tb:TaskbarIcon.DataContext>
    </tb:TaskbarIcon>
</ResourceDictionary>
```

The context menu is defined between the `<ContextMenu>` tags with a `<Separator />` and some commands. The icon can be referenced in the `tb:TaskbarIcon>` tag, alongside the dataContext and further commands for UI control.


***Changing Icon on events, e.g. click***

If you want to have a changing icon for certain events, e.g. grey icon for a deactivated mode and green icon for activated mode, you can look at my [Winsomnia](https://github.com/Skjoldrun/Winsomnia) project, where I have implemented exactly that.

With the class `NotifyIcon` I have built the base for accessing and initializing the NotifyIcon on app startup:

```csharp
using Hardcodet.Wpf.TaskbarNotification;

namespace Winsomnia.Utility
{
    public static class NotifyIcon
    {
        public static TaskbarIcon TrayIcon;
    }
}
```

This gets used in the App.xaml.cs:

```csharp
using Hardcodet.Wpf.TaskbarNotification;
using System.Windows;
using Winsomnia.Utility;
using Winsomnia.ViewModel;

namespace Winsomnia
{
    /// <summary>
    /// Interaction logic for App.xaml
    /// </summary>
    public partial class App : Application
    {
        protected override void OnStartup(StartupEventArgs e)
        {
            base.OnStartup(e);

            NotifyIcon.TrayIcon = (TaskbarIcon)FindResource("NotifyIcon");

            var notifyIconVM = new NotifyIconViewModel();
            NotifyIcon.TrayIcon.DataContext = notifyIconVM;

            if (Winsomnia.Properties.Settings.Default.ActivateOnStart)
                notifyIconVM.SwitchMode();
        }

        protected override void OnExit(ExitEventArgs e)
        {
            NotifyIcon.TrayIcon.Dispose();
            base.OnExit(e);
        }
    }
}
```

The ressource for it is a bit smaller than the example on the top of the article:

```xml
<tb:TaskbarIcon x:Key="NotifyIcon"
    IconSource="/Resource/Default.ico"
    ToolTipText="Winsomnia"
    LeftClickCommand="{Binding SwitchModeCommand}"
    DoubleClickCommand="{Binding SwitchModeCommand}"
    ContextMenu="{StaticResource SysTrayMenu}">
</tb:TaskbarIcon>
```

The switch of the system mode should also switch the icon, either with a call from the LeftClickCommand or context menu, or else. This can be achieved with accessing the NotifyIcon in the ViewModel and replace the defined icon with another one:

![StandUpMate Systemtray Icon](/assets/images/coding/wpf/systemtray-icon/change-icon.png)


