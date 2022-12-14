---
layout: page
title: WPF - commands
parent: WPF
---

# Commands

***Youtube Tutorials:** [Commands Part I](https://www.youtube.com/watch?v=HDSRG7GvPbo) and [Commands Part II](https://www.youtube.com/watch?v=8WfD2cFRymM)*

Commands enable a bound method from the ViewModel to be executet from a View element, like a button. You can add an execution validation if the UI element should be shown as diasabled for preventing the execution (e.g. you have to enter data first to execute a save command). 

A possible implementation of a application wide Command base class could be the following:

```csharp
using System;
using System.Windows.Input;
namespace StandUpMate.Command
{
    public class DelegateCommand : ICommand
    {
        public Action CommandAction { get; set; }
        public Func<bool> CanExecuteFunc { get; set; }

        public void Execute(object parameter)
        {
            CommandAction();
        }

        public bool CanExecute(object parameter)
        {
            return CanExecuteFunc == null || CanExecuteFunc();
        }

        public event EventHandler CanExecuteChanged
        {
            add { CommandManager.RequerySuggested += value; }
            remove { CommandManager.RequerySuggested -= value; }
        }
    }
}
```

This can be then used linke in the following example, where I want to open a settings window, but only if it is not allready opend yet:

```csharp
/// <summary>
/// Shows the settings window if not opened yet.
/// </summary>
public ICommand ShowSettingsWindowCommand
{
    get
    {
        return new DelegateCommand
        {
            CanExecuteFunc = () => Application.Current.MainWindow == null 
	                           || Application.Current.MainWindow.IsActive == false,
            CommandAction = () =>
            {
                Application.Current.MainWindow = new MainWindow();
                Application.Current.MainWindow.Show();
            }
        };
    }
}
```
With `CanExecuteFunc` I validate if the window is already open. The `CommandAction` then executes the part to open the window.

This code is from my [StandUpMate](https://github.com/Skjoldrun/StandUpMate) project.


## Commands with multiple parameters

Besides simple binding metheods to View elements, you can also process multiple parameters through the command call. To acheive this you have to build a MultiValueConverter to pack the parameters:

```csharp
/// <summary>
/// Converts multiple command parameters to pass as one.
/// </summary>
public class ArrayMultiValueConverter : IMultiValueConverter
{
    public object Convert(object[] values, Type targetType, object parameter, CultureInfo culture)
    {
        return values.Clone();
    }
    public object[] ConvertBack(object value, Type[] targetTypes, object parameter, CultureInfo culture)
    {
        throw new NotImplementedException();
    }
}
```

This has to be referenced in your XAML code (App.xaml):

```xml
<Application.Resources>
    <u:ArrayMultiValueConverter x:Key="ArrayMultiValueConverter" />
</Application.Resources>
```

[![ArrayMultiValueConverter refference in XAML](/assets/images/coding/wpf/commands/ArrayMultiValueConverter-reference.png)](/assets/images/coding/wpf/commands/ArrayMultiValueConverter-reference.png)

Now you can send multiple parameters from UI elements with the click of a button:

[![ArrayMultiValueConverter refference in XAML](/assets/images/coding/wpf/commands/example-view.png)](/assets/images/coding/wpf/commands/example-view.png)

This is how you access the parameters in the ViewModel:

[![ArrayMultiValueConverter refference in XAML](/assets/images/coding/wpf/commands/access-param-ViewModel.png)](/assets/images/coding/wpf/commands/access-param-ViewModel.png)
