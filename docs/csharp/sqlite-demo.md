---
layout: page
title: C# - SQLite Demo
parent: C#
---

# SQLite Demo

## Intro

SQLite is a very useful way to have a local small DB system to cache or store relative data for your application. You get a fully functioning SQL DB System to be used by a DB ORM tool like Dapper or EF, without having a heavy system to install or to run besides your application. The SQLite DB lies as file besides your application and therefore is super portable as well.

A good way to work and create SQLite DB is the tool [DB Browser for SQLite](https://sqlitebrowser.org/).


## Creating a Demo C# WinForm

I have created a small demo with WinForms and Dapper to show some code and the use of the SQLite DB. The demo project contains an UI with simple text boxes and a list box:

![UI editor](/assets/images/coding/csharp/sqlite-demo/ui-editor.png)

The list box is bound to a list of type `PersonModel` to be able to show some DB data. The Buttons are wired up in the Code-behind for this simple demo and call the `SqliteDataAccess` class and its DB access methods with Dapper.

You have to install some NuGet packages to work with SQLite and Dapper: `Dapper` and `System.Data.SQLite.Core`.

The `SqliteDataAccess` class then opens a connection to the DB and sends the SQL statements.

**SqliteDataAccess**

```csharp
public class SqliteDataAccess
{
    public static List<PersonModel> LoadPeople()
    {
        using (IDbConnection cnn = new SQLiteConnection(LoadConnectionString()))
        {
            var output = cnn.Query<PersonModel>("SELECT * FROM Person", new DynamicParameters());
            return output.ToList();
        }
    }

    public static void SavePerson(PersonModel person)
    {
        using (IDbConnection cnn = new SQLiteConnection(LoadConnectionString()))
        {
            cnn.Execute("INSERT INTO Person (FirstName, LastName) VALUES (@FirstName, @LastName)", person);
        }
    }

    public static void DeletePerson(PersonModel person)
    {
        using (IDbConnection cnn = new SQLiteConnection(LoadConnectionString()))
        {
            cnn.Execute("DELETE FROM Person WHERE FirstName = @FirstName AND LastName = @LastName", person);
        }
    }

    public static List<PersonModel> SelectWithHackable(string searchString)
    {
        using (IDbConnection cnn = new SQLiteConnection(LoadConnectionString()))
        {
            var sqlStatement = "SELECT * FROM Person WHERE LastName = '" + searchString + "';";
            var output = cnn.Query<PersonModel>(sqlStatement);
            return output.ToList();
        }
    }

    private static string LoadConnectionString(string connectionStringName = "Default")
    {
        return ConfigurationManager.ConnectionStrings[connectionStringName].ConnectionString;
    }
}
``` 

The retrieved data are list with type `PersonModel`.

**PersonModel**

```csharp
public class PersonModel
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }

    public string FullName
    {
        get
        {
            return $"{FirstName} {LastName}";
        }
    }
}
``` 

With the code in the UI you can add new persons to the DB, read the DB, Delete entries and even try to hack the DB access with a small SQL injection via the `?`button and a LastName text injection. This is possible because the used SQL statement behind the button does not get secured by parameterized SQL, but the SQL get build with string concatenation.

**UI Code-behind**

```csharp
public partial class Main : Form
{
    private List<PersonModel> _people = new List<PersonModel>();

    public Main()
    {
        InitializeComponent();
        LoadPeopleList();
    }

    private void LoadPeopleList()
    {
        _people = SqliteDataAccess.LoadPeople();

        WireUpListBox();
    }

    private void WireUpListBox()
    {
        listPeopleListBox.DataSource = null;
        listPeopleListBox.DataSource = _people;
        listPeopleListBox.DisplayMember = "FullName";
        listPeopleListBox.ValueMember = "Id";
    }

    private void btnAddPerson_Click(object sender, EventArgs e)
    {
        if (ValidateTextboxInput() == false)
            return;

        PersonModel p = new PersonModel();
        p.FirstName = firstNameText.Text;
        p.LastName = lastNameText.Text;

        SqliteDataAccess.SavePerson(p);

        firstNameText.Text = "";
        lastNameText.Text = "";

        LoadPeopleList();
    }

    private bool ValidateTextboxInput()
    {
        if (string.IsNullOrWhiteSpace(firstNameText.Text)
                        || string.IsNullOrWhiteSpace(lastNameText.Text))
        {
            MessageBox.Show(
                "FirstName and LastName must contain text.",
                "Invalid Entries",
                MessageBoxButtons.OK,
                MessageBoxIcon.Error);
            return false;
        }

        return true;
    }

    private void btnRefresh_Click(object sender, EventArgs e)
    {
        LoadPeopleList();
    }

    private void btnDelete_Click(object sender, EventArgs e)
    {
        var selectedItem = listPeopleListBox.SelectedItem;

        // This just gets the listPeopleListBox.ValueMember, therefore the Id value
        //var selectedValue = listPeopleListBox.SelectedValue;

        SqliteDataAccess.DeletePerson(selectedItem as PersonModel);

        LoadPeopleList();
    }

    private void btnHack_Click(object sender, EventArgs e)
    {
        if (ValidateTextboxInput() == false)
            return;

        var resultList = SqliteDataAccess.SelectWithHackable(lastNameText.Text);

        var result = new StringBuilder();
        resultList.ForEach(p => result.Append(p.FullName + " "));

        MessageBox.Show(result.ToString());

        firstNameText.Text = "";
        lastNameText.Text = "";

        LoadPeopleList();
    }
}
``` 
