---
layout: page
title: WPF - UI Theme MahApps.Metro
parent: WPF
---

# UI Theme MahApps.Metro

WPF Views can be themed to look way more interesting than the default look. To achieve this you can use the UI Theme [MahApps.Metro](https://github.com/MahApps/MahApps.Metro).


## Preview

[![MahApps Preview](/assets/images/articles/theme-mahApps/preview.png)](/assets/images/articles/theme-mahApps/preview.png)


## Quick Start

Theres a quick start guide on from MahApps: [Wiki QuickStart](https://github.com/MahApps/MahApps.Metro/wiki/Quick-Start) or [Guides QuickStart](https://mahapps.com/docs/guides/quick-start).


**Install the Nuget Package**

[![MahApps Preview](/assets/images/articles/theme-mahApps/nuget-package.png)](/assets/images/articles/theme-mahApps/nuget-package.png)


**Add ressources in App.xaml**

[![MahApps Preview](/assets/images/articles/theme-mahApps/ressources-app.xaml.png)](/assets/images/articles/theme-mahApps/ressources-app.xaml.png)

```xml
<Application x:Class="WpfApplication.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             StartupUri="MainWindow.xaml">
  <Application.Resources>
    <ResourceDictionary>
      <ResourceDictionary.MergedDictionaries>
        <ResourceDictionary Source="pack://application:,,,/MahApps.Metro;component/Styles/Controls.xaml" />
        <ResourceDictionary Source="pack://application:,,,/MahApps.Metro;component/Styles/Fonts.xaml" />
        <ResourceDictionary Source="pack://application:,,,/MahApps.Metro;component/Styles/Themes/Light.Blue.xaml" />
      </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
  </Application.Resources>
</Application>
```

**Change View to MahApps Window**

Change the `Window` to the `mah:MetroWindow`and add the `xmlns:mah="http://metro.mahapps.com/winfx/xaml/controls"` as namespace:

[![MahApps Preview](/assets/images/articles/theme-mahApps/mahApp-window-View.png)](/assets/images/articles/theme-mahApps/mahApp-window-View.png)

```xml
<mah:MetroWindow x:Class="HelloWorldWPF.View.HelloWorldView"
                 xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                 xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                 xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
                 xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
                 xmlns:mah="clr-namespace:MahApps.Metro.Controls;assembly=MahApps.Metro"
                 xmlns:local="clr-namespace:HelloWorldWPF.View"
                 xmlns:vm="clr-namespace:HelloWorldWPF.ViewModel"
                 mc:Ignorable="d"
                 Title="HelloWorldView" Height="200" Width="300"
                 WindowStartupLocation="CenterScreen">
    <Window.DataContext>
        <vm:HelloWorldViewModel />
    </Window.DataContext>
    <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="20" />
            <ColumnDefinition Width="*" />
            <ColumnDefinition Width="20" />
        </Grid.ColumnDefinitions>
        <Grid.RowDefinitions>
            <RowDefinition Height="20" />
            <RowDefinition Height="auto" />
            <RowDefinition Height="auto" />
            <RowDefinition Height="*" />
            <RowDefinition Height="auto" />
            <RowDefinition Height="20" />
        </Grid.RowDefinitions>
        <Label Grid.Column="1"
               Grid.Row="1"
               x:Name="label"
               FontSize="30"
               Margin="0"
               Content="{Binding HelloString}"
               HorizontalAlignment="Center"
               VerticalAlignment="Center" />
        <Label Grid.Column="1"
               Grid.Row="2"
               x:Name="labelTheme"
               FontSize="20"
               Margin="0 5 0 5"
               Content="{Binding ActiveTheme}"
               HorizontalAlignment="Center"
               VerticalAlignment="Center" />
        <UniformGrid Grid.Column="1" Grid.Row="3" Columns="2">
            <Button Grid.Column="1"
                    Height="30"
                    Content="Swap Themes"
                    x:Name="btnSwapUiTheme"
                    Click="btnSwapUiTheme_Click" />
            <Button Grid.Column="1"
                    Height="30"
                    Content="Do something"
                    x:Name="btnDoSomething"
                    Command="{Binding CmdDoSomething}" />
        </UniformGrid>
    </Grid>
</mah:MetroWindow>
```

Add the inheritance of the `MetroWindow` class in the Code-behind (optional, not always necessary):

[![MahApps Preview](/assets/images/articles/theme-mahApps/Code-behind.png)](/assets/images/articles/theme-mahApps/Code-behind.png)

```csharp
using ControlzEx.Theming;
using MahApps.Metro.Controls;
using MahApps.Metro.Controls.Dialogs;
using System;
using System.Collections.Generic;
using System.Windows;
namespace HelloWorldWPF.View
{
    /// <summary>
    /// Interaktionslogik für HelloWorldView.xaml
    /// </summary>
    public partial class HelloWorldView : MetroWindow
    {
        #region Fields
        /// <summary>
        /// Dictionary with keyValue pairs for MahApp.Metro Themes. 
        /// </summary>
        private Dictionary<int, string> themes = new Dictionary<int, string>()
            {
                {1, "Red"},
                {2, "Green"},
                {3, "Blue"},
                {4, "Purple"},
                {5, "Orange"},
                {6, "Lime"},
                {7, "Emerald"},
                {8, "Teal"},
                {9, "Cyan"},
                {10, "Cobalt"},
                {12, "Indigo"},
                {13, "Violet"},
                {14, "Pink"},
                {15, "Magenta"},
                {16, "Crimson"},
                {17, "Amber"},
                {18, "Yellow"},
                {19, "Brown"},
                {20, "Sienna"},
                {21, "Olive"},
                {22, "Steel"},
                {23, "Mauve"},
                {24, "Taupe"}
            };
        #endregion Fields

        #region Constructor
        /// <summary>
        /// Constructor inits the View and sets the MahApp.Metro Theme with synced Themes for all Windows.
        /// </summary>
        public HelloWorldView()
        {
            InitializeComponent();
            ThemeManager.Current.ChangeTheme(this, "Dark.Blue");
            ThemeManager.Current.ThemeSyncMode = ThemeSyncMode.SyncWithAppMode;
            ThemeManager.Current.SyncTheme();
        }
        #endregion Constructor
        
        #region Methods
        /// <summary>
        /// Click event randomizes the MahApp.Metro Theme with a random key for the themes dictionary.
        /// </summary>
        private void btnSwapUiTheme_Click(object sender, RoutedEventArgs e)
        {
            Random random = new Random();
            int themeKey = random.Next(1, themes.Count);
            if (themes.ContainsKey(themeKey))
            {
                ThemeManager.Current.ChangeTheme(this, $"Dark.{themes[themeKey]}");
                labelTheme.Content = $"Dark.{themes[themeKey]}";
            }
            else
            {
                this.ShowMessageAsync("Error", $"Random themeKey {themeKey} was not in the themepack!");
            }
        }
        #endregion Methods
    }
}
```


## some test with a ViewModel

[![MahApps Preview](/assets/images/articles/theme-mahApps/preview.gif)](/assets/images/articles/theme-mahApps/preview.gif)


**ViewModel**

```csharp
using ControlzEx.Theming;
using HelloWorldWPF.Commands;
using HelloWorldWPF.Model;
using System;
using System.ComponentModel;
using System.Runtime.CompilerServices;
using System.Windows.Input;
namespace HelloWorldWPF.ViewModel
{
    public class HelloWorldViewModel : INotifyPropertyChanged
    {
        #region Fields
        private string helloString;
        private string activeTheme;
        private ClientCommand cmdDoSomething;   // use custom implementation of ClientCommand
        public event PropertyChangedEventHandler PropertyChanged;
        #endregion Fields

        #region Properties
        public string HelloString
        {
            get
            {
                return helloString;
            }
            set
            {
                helloString = value;
                OnPropertyChanged();
            }
        }
        public string ActiveTheme
        {
            get
            {
                return activeTheme;
            }
            set
            {
                activeTheme = value;
                OnPropertyChanged();
            }
        }
        public ICommand CmdDoSomething
        {
            get
            {
                if (cmdDoSomething == null)
                {
                    cmdDoSomething = new ClientCommand(CmdDoSomethingExecute);
                }
                return cmdDoSomething;
            }
        }
        #endregion Properties

        #region Constructor
        /// <summary>
        /// Constructor with object of the model and init of the properties.
        /// </summary>
        public HelloWorldViewModel()
        {
            HelloWorldModel helloWorldModel = new HelloWorldModel();
            helloString = helloWorldModel.ImportantInfo;
            ActiveTheme = ThemeManager.Current.DetectTheme().Name;
        }
        #endregion Constructor

        #region Methods
        /// <summary>
        /// Raises OnPropertychangedEvent when property changes.
        /// </summary>
        /// <param name="name">String representing the property name</param>
        protected void OnPropertyChanged([CallerMemberName] string name = null)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
        }
        /// <summary>
        /// Command execution simulation with some code that changes a property and the UI shown value.
        /// </summary>
        /// <param name="commandParameter"></param>
        private void CmdDoSomethingExecute(object commandParameter)
        {
            ActiveTheme = $"changed: {DateTime.Now.ToString("dd.MM.yyyy-HH:mm:ss")}";
        }
        #endregion Methods
    }
}
```


**Model**

```csharp
using System.Collections.Generic;
using System.Linq;
namespace HelloWorldWPF.Model
{
    public class HelloWorldModel
    {
        #region Fields
        private List<string> repositoryData;
        #endregion Fields
        #region Properties
        public string ImportantInfo
        {
            get
            {
                return ConcatenateData(repositoryData);
            }
        }
        #endregion Properties

        #region Constructor
        /// <summary>
        /// Constructor for the model simulates a repository data access ...
        /// </summary>
        public HelloWorldModel()
        {
            repositoryData = GetData();
        }
        #endregion Constructor

        #region Methods
        /// <summary>
        /// Simulates data retrieval from a repository
        /// </summary>
        /// <returns>List of strings</returns>
        private List<string> GetData()
        {
            repositoryData = new List<string>()
            {
                "Hello",
                "world"
            };
            return repositoryData;
        }
        /// <summary>
        /// Concatenate the information from the list into a fully formed sentence.
        /// </summary>
        /// <returns>A string</returns>
        private string ConcatenateData(List<string> dataList)
        {
            string importantInfo = dataList.ElementAt(0) + ", " + dataList.ElementAt(1) + "!";
            return importantInfo;
        }
        #endregion Methods
    }
}
```

**ClientCommand class**

The ClientCommand class enables binding methods from the VieModel into the View. This represents a reusable base class to setup the needed functionality and adds the possibility to deactivate the command in configured situations. You can disable the bound UI element with this as well.

```csharp
using System;
using System.Windows.Input;
namespace HelloWorldWPF.Commands
{
    public class ClientCommand : ICommand
    {
        #region Fields
        private readonly Predicate<object> _CanExecutePredicate;
        private readonly Action<object> _ExecuteAction;
        #endregion Fields

        #region Constructors
        public ClientCommand(Action<object> executeAction)
        {
            _ExecuteAction = executeAction;
        }
        public ClientCommand(Action<object> executeAction, Predicate<object> canExecutePredicate)
        {
            _ExecuteAction = executeAction;
            _CanExecutePredicate = canExecutePredicate;
        }
        #endregion Constructors

        #region Methods
        public event EventHandler CanExecuteChanged
        {
            add
            {
                CommandManager.RequerySuggested += value;
            }
            remove
            {
                CommandManager.RequerySuggested -= value;
            }
        }
        public bool CanExecute(object parameter)
        {
            return _CanExecutePredicate == null || _CanExecutePredicate(parameter);
        }
        public void Execute(object parameter)
        {
            _ExecuteAction(parameter);
        }
        #endregion Methods
    }
}
```
