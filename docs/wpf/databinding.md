---
layout: page
title: WPF - databinding
parent: WPF
---

# Databinding in Views

Databinding connects datavalues in Views (e.g. TextBox entries) to Properties of the corresponding ViewModel behind the View. If the get connected and configured properly, you can bind their data to be updated if the ViewModel value changes through a method, or update the ViewModel Property if the user changes the data via UI.


## Connect View and ViewModel

The View and the ViewModel have to be connected as DataContext, e.g. via DataContext in the View XAML:

[![datacontext](/assets/images/articles/data-binding/datacontext.png)](/assets/images/articles/data-binding/datacontext.png)

If the connection works you often have IntelliSense support in the XAML editor, to access the properties of the ViewModel:

[![datacontext](/assets/images/articles/data-binding/intelliSense.png)](/assets/images/articles/data-binding/intelliSense.png)


## Binding

The binding happens in the View. You can bind View elements and their values to ViewModel properties in certain modes:

| Mode              | Description                                                             |
| ----------------- | ----------------------------------------------------------------------- |
| One-Way           | Updates in one way (source to destination)                              |
| Two-Way           | Updates in both ways                                                    |
| One-Way to source | Updates in one way (destination to source)                              |
| One-Time          | Updates only once                                                       |
| Default           | Differs for every UI element (TextBlock is One-Way, TextBox is Two-Way) |

*Source: [s-sharpcorner](https://www.c-sharpcorner.com/article/data-binding-its-modes-in-wpf/)*


## Example

```xml
<TextBox    
Grid.Column="1" Grid.Row="2" 
Margin="5" MinWidth="150" 
x:Name="txtVar01" 
Text="{Binding WriteValue01, UpdateSourceTrigger=PropertyChanged}" />
```

The default mode on TextBoxes is Two-Way, the value is bound to the property `WriteValue01`. With `UpdateSourceTrigger=PropertyChanged`, the update gets called on change events. This property is implemented with a `OnPropertyChanged()` method in the ViewModel:

```csharp
public string WriteValue01
{
    get { return writeValue01; }
    set
    {
        writeValue01 = value;
        OnPropertyChanged();
    }
}
```

The `OnPropertyChanged()` method takes care of the value updates. This can be implemented with a parent class, called `ObservableObject `, which gets inherited by the ViewModel:

```csharp
/// <summary>
/// Implementation of the INotifyPropertyChanged for multiple classes which inherit from this.
/// </summary>
public class ObservableObject : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler PropertyChanged;
    /// <summary>
    /// Raises OnPropertychangedEvent when property changes.
    /// </summary>
    /// <param name="name">String representing the property name</param>
    protected void OnPropertyChanged([CallerMemberName] string name = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
    }
}
```

The `INotifyPropertyChanged` interface is the .NET base interface for this mechanism.

***Tutorial on YouTube:** [Property Changes](https://www.youtube.com/watch?v=LEKngPq342s)*