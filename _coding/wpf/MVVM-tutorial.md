---
layout: page
title: WPF - MVVM tutorial
---

# MVVM Tutorial

*I took this tutorial from [riptutorial](https://riptutorial.com/mvvm/example/15507/csharp-mvvm-summary-and-complete-example).*

## App.xaml

Add hte MVVM folder structure in the WPF project:

![folder structure](/assets/images/coding/wpf/MVVM-Tutorial/Folder-structure.png)

The StartUpUri points to the entry point of the project and to which View should be opened.


## ViewModel

The ViewModel accesses the Model and implements a interface to recognize and handle changes `INotifyPropertyChanged`. 

![ViewModel](/assets/images/coding/wpf/MVVM-Tutorial/ViewModel.png)

```csharp
using HelloWorldWPF.Model;
using System.ComponentModel;
using System.Runtime.CompilerServices;

namespace HelloWorldWPF.ViewModel
{
    public class HelloWorldViewModel : INotifyPropertyChanged
    {
        private string helloString;

        public event PropertyChangedEventHandler PropertyChanged;

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

        public HelloWorldViewModel()
        {
            HelloWorldModel helloWorldModel = new HelloWorldModel();
            helloString = helloWorldModel.ImportantInfo;
        }

        /// <summary>
        /// Raises OnPropertychangedEvent when property changes
        /// </summary>
        /// <param name="name">String representing the property name</param>
        protected void OnPropertyChanged([CallerMemberName] string name = null)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
        }
    }
}
```

The Method `OnPropertyChanged(...)` communicates change events. With this the ViewModel can communicate changes to the View and vice versa.


## Model

The Model has access to the Repository (if the Repository Pattern is implemented, here its just simulated). The Repository gets initiated in the Constructor.

![Model](/assets/images/coding/wpf/MVVM-Tutorial/Model.png)

```csharp
using System.Collections.Generic;
using System.Linq;

namespace HelloWorldWPF.Model
{
    public class HelloWorldModel
    {
        private List<string> repositoryData;

        public string ImportantInfo
        {
            get
            {
                return ConcatenateData(repositoryData);
            }
        }

        public HelloWorldModel()
        {
            repositoryData = GetData();
        }

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
    }
}
```


## View

THe View has the design and UI elements and could have some Code-behind, too. For MVVM, you only ever should add code -behind, if this only has to do with the View itself, but never to control, or manipulate data or other parts of the program. Such implementation always should be in the ViewModels to keep the View independent from the lower levels.

Views are written in XAML and reference their ViewModel as DataContext. Therefore the Namespace gets registered with `xmlns:vm="clr-namespace:MyMVVMProject.ViewModel"` and a linking with `<Window.DataContext><vm:HelloWorldViewModel/></Window.DataContext>` is one version of connecting the ViewModel with the Model.
The DataContext can also be connected in the Code-Behind with `DataContext = new HelloWorldViewModel();` or in ressource tables in the App.xml.

![View](/assets/images/coding/wpf/MVVM-Tutorial/View.png)

```csharp
<Window x:Class="HelloWorldWPF.View.HelloWorldView"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:HelloWorldWPF.View"
        xmlns:vm="clr-namespace:HelloWorldWPF.ViewModel"
        mc:Ignorable="d"
        Title="HelloWorldView" Height="150" Width="300">
    <Window.DataContext>
        <vm:HelloWorldViewModel />
    </Window.DataContext>
    <Grid>
        <Label x:Name="label" FontSize="30" Content="{Binding HelloString}" 
               HorizontalAlignment="Center" VerticalAlignment="Center" />
    </Grid>
</Window>
```