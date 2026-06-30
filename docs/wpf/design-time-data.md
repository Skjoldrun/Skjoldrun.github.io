---
layout: page
title: WPF - Design-time data
parent: WPF
---

# Design-time data in WPF with `d:DesignInstance`

When you build WPF Views with the MVVM pattern, the real `DataContext` is usually assigned at runtime (e.g. through dependency injection or a `DataTemplate`). That is great for the running application, but the Visual Studio / Blend XAML designer has *no idea* what type sits behind your bindings. The result is an empty, lifeless design surface and no IntelliSense for your `{Binding}` paths.

This tutorial shows how to feed the designer with **design-time data** using `d:DataContext` and `{d:DesignInstance}`, so you get a populated preview and full binding IntelliSense — without affecting the runtime behaviour of your app.


## The goal

We want to add a single line to a View that does two things:

1. Tell the designer which ViewModel type the View binds to (IntelliSense for binding paths).
2. Show realistic sample data on the design surface (text, list items, states, etc.).

The end result looks like this:

```xml
<Window x:Class="YOURAPP.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:dt="clr-namespace:YOURAPP.DesignTime"
        mc:Ignorable="d"
        d:DataContext="{d:DesignInstance Type=dt:DesignMainWindowViewModel, IsDesignTimeCreatable=True}"
        Title="MainWindow" Height="450" Width="800">
    ...
</Window>
```


## How it works: the `d:` namespace and `mc:Ignorable`

Two XML namespaces are key here:

```xml
xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
mc:Ignorable="d"
```

- The `d:` prefix is the **design** namespace. Anything prefixed with `d:` only exists for the XAML designer.
- `mc:Ignorable="d"` tells the XAML parser (and the runtime) that the `d:` prefixed attributes can be safely **ignored** when the app actually runs.

That is the whole trick: `d:DataContext` overrides the design surface only, while the real `DataContext` is still set at runtime. The two never collide.

| Attribute        | Active at design time | Active at runtime |
| ---------------- | --------------------- | ----------------- |
| `DataContext`    | ❌ (usually unset)     | ✅                 |
| `d:DataContext`  | ✅                     | ❌ (ignored)       |


## `{d:DesignInstance}` explained

`{d:DesignInstance}` is a markup extension that creates (or fakes) an instance of a type for the designer:

```xml
d:DataContext="{d:DesignInstance Type=dt:DesignMainWindowViewModel, IsDesignTimeCreatable=True}"
```

- **`Type`** — the type the designer should treat as the `DataContext`. The designer reads its public properties to provide binding IntelliSense.
- **`IsDesignTimeCreatable`** — controls *how* the designer obtains the data:
  - `True` → the designer **actually instantiates** the type (calls its parameterless constructor) and shows the real values your design-time ViewModel exposes. Use this when you want to *see sample data*.
  - `False` (default) → the designer does **not** create an instance. It only inspects the type's shape for IntelliSense and shows empty placeholders. Use this when you just want binding autocompletion.

For a populated preview you almost always want `IsDesignTimeCreatable=True` together with a dedicated design-time ViewModel.


## Step-by-step tutorial

### Step 1 — Create a DesignTime folder/namespace

Keep your design-time helpers separate from production code. Create a folder `DesignTime` in your project so the namespace becomes `YOURAPP.DesignTime`:

```
YOURAPP/
├── ViewModels/
│   └── MainWindowViewModel.cs
├── DesignTime/
│   └── DesignMainWindowViewModel.cs
└── Views/
    └── MainWindow.xaml
```

### Step 2 — The real ViewModel

This is your normal, runtime ViewModel. It typically needs services injected through its constructor:

```csharp
namespace YOURAPP.ViewModels
{
    public class MainWindowViewModel : ObservableObject
    {
        private readonly IUserService userService;

        public MainWindowViewModel(IUserService userService)
        {
            this.userService = userService;
            Users = new ObservableCollection<UserItem>(userService.GetUsers());
        }

        public string Title { get; set; }
        public ObservableCollection<UserItem> Users { get; }
        public bool IsBusy { get; set; }
    }
}
```

Because this constructor takes an `IUserService`, the designer cannot create it directly — which is exactly why we add a design-time variant.

### Step 3 — The design-time ViewModel

Create a ViewModel that **inherits from** (or mirrors) the real one and has a **parameterless constructor** filled with believable sample data:

```csharp
using System.Collections.ObjectModel;
using YOURAPP.ViewModels;

namespace YOURAPP.DesignTime
{
    /// <summary>
    /// Design-time only ViewModel that provides sample data for the XAML designer.
    /// It is never used at runtime.
    /// </summary>
    public class DesignMainWindowViewModel : MainWindowViewModel
    {
        public DesignMainWindowViewModel()
            : base(new DesignUserService())
        {
            Title = "Design-time preview";
            IsBusy = false;

            Users.Clear();
            Users.Add(new UserItem { Name = "Ada Lovelace",  Role = "Admin"  });
            Users.Add(new UserItem { Name = "Alan Turing",   Role = "User"   });
            Users.Add(new UserItem { Name = "Grace Hopper",  Role = "Editor" });
        }
    }
}
```

> **Tip:** If inheriting is awkward (e.g. the base constructor does heavy work), you can instead create a *standalone* design ViewModel that simply exposes the same public properties the View binds to. The designer only cares about matching property names and types.

If your real ViewModel needs a service, provide a tiny fake for design time:

```csharp
namespace YOURAPP.DesignTime
{
    internal class DesignUserService : IUserService
    {
        public IEnumerable<UserItem> GetUsers() => Array.Empty<UserItem>();
    }
}
```

### Step 4 — Wire it up in the View

Add the `dt` namespace and the `d:DataContext` to your `Window` (or `UserControl`):

```xml
<Window x:Class="YOURAPP.Views.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:dt="clr-namespace:YOURAPP.DesignTime"
        mc:Ignorable="d"
        d:DataContext="{d:DesignInstance Type=dt:DesignMainWindowViewModel, IsDesignTimeCreatable=True}"
        Title="MainWindow" Height="450" Width="800">

    <Grid Margin="10">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto" />
            <RowDefinition Height="*" />
        </Grid.RowDefinitions>

        <TextBlock Grid.Row="0"
                   FontSize="20"
                   Text="{Binding Title}" />

        <DataGrid Grid.Row="1"
                  Margin="0,10,0,0"
                  ItemsSource="{Binding Users}"
                  AutoGenerateColumns="False">
            <DataGrid.Columns>
                <DataGridTextColumn Header="Name" Binding="{Binding Name}" />
                <DataGridTextColumn Header="Role" Binding="{Binding Role}" />
            </DataGrid.Columns>
        </DataGrid>
    </Grid>
</Window>
```

As soon as you build the project, the designer instantiates `DesignMainWindowViewModel` and the `DataGrid` shows *Ada Lovelace*, *Alan Turing* and *Grace Hopper* — all without running the app.

### Step 5 — Keep the runtime DataContext separate

The real `DataContext` is still assigned at runtime, e.g. in code-behind or via a DI container:

```csharp
public partial class MainWindow : Window
{
    public MainWindow(MainWindowViewModel viewModel)
    {
        InitializeComponent();
        DataContext = viewModel; // runtime DataContext
    }
}
```

Because `d:DataContext` is ignored at runtime (`mc:Ignorable="d"`), there is no conflict between the design-time and runtime contexts.


## Design-time-only data inside the View

Besides the whole `DataContext`, you can also set **individual** design-time values with `d:` prefixed properties. These are handy for previewing a single element:

```xml
<!-- Shows "Sample title" only in the designer -->
<TextBlock Text="{Binding Title}" d:Text="Sample title" />

<!-- Preview list items without a design ViewModel -->
<ListBox d:ItemsSource="{d:SampleData ItemCount=5}" />
```

`d:Text`, `d:Visibility`, `d:IsEnabled`, etc. all override the runtime value on the design surface only.


## Benefits

- **Visual feedback while you design** — see realistic content (lists, text, states) instead of an empty layout, so spacing, column widths and templates can be judged immediately.
- **Binding IntelliSense** — typing `{Binding ...}` offers the properties of the design type, which reduces typos and broken bindings.
- **Catch binding errors early** — a misspelled property is visible in the designer instead of only showing up as a silent runtime binding failure.
- **No runtime impact** — everything lives behind the `d:` namespace and is stripped out by `mc:Ignorable="d"`. No extra code runs, no extra memory is used in production.
- **Designer-friendly even with DI** — your real ViewModel can require constructor-injected services; the design ViewModel provides a parameterless, data-filled alternative.
- **Works in Blend and Visual Studio** — both use the same `d:` design namespace.


## Common pitfalls

| Problem | Cause / Fix |
| ------- | ----------- |
| Designer shows nothing | You set `IsDesignTimeCreatable=True` but the design type has **no parameterless constructor**, or its constructor throws. Add a safe parameterless constructor. |
| `dt` namespace not found | The `clr-namespace:YOURAPP.DesignTime` value must match the **actual namespace**, and the type must be in the **same assembly** (otherwise add `;assembly=YourAssembly`). |
| Sample data appears at runtime | You used `DataContext` instead of `d:DataContext`, or forgot `mc:Ignorable="d"`. |
| IntelliSense works but no data | You used `IsDesignTimeCreatable=False` (or omitted it). Set it to `True` for a created instance. |
| Changes not visible | Rebuild the project — the designer needs a compiled assembly to instantiate the design ViewModel. |


## Summary

Design-time data lets you develop WPF Views with a populated, IntelliSense-aware designer while keeping the runtime `DataContext` completely separate:

1. Add the `d:` and `mc:` namespaces and `mc:Ignorable="d"`.
2. Create a `YOURAPP.DesignTime` namespace with a parameterless design ViewModel full of sample data.
3. Point the View at it with `d:DataContext="{d:DesignInstance Type=dt:DesignMainWindowViewModel, IsDesignTimeCreatable=True}"`.
4. Assign the real ViewModel at runtime as usual.

The `d:` namespace guarantees all of this is design-time only — your shipping application stays untouched.
