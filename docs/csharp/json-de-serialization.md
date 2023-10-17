---
layout: page
title: C# - JSON (de)serialization 
parent: C#
---

# JSON (de)serialization 

JSON files are great for configurations, datastorage, mappings, etc. Reading and writing objects from and to JSON is called deserialization and serialization. This can be done with the `System.Text.JSON` NuGet package.

***Note:** The Newtonsoft package is wildly used, but I've heard the main developer behind Newtonsoft has changed to MS and now develops the `System.Text.JSON` package and this gets replaced more and more.*


## JSON Structure and C# Mapping class

The JSON structure can be given by the C# class, wich defines how the JSON will be written. But if you have a JSON file and structure already defined, you need to build the mathcing C# class to be able to deserialize the data from JSON to an C# object.


### Deserialization Example

Deserialize a given JSON file with fictional machines and their properties:

```json
{
  "Machines": [
    {
      "Id": 1,
      "Name": "Lasercarver 1000",
      "MachineType": 3,
      "Integrity": 1.0
    },
    {
      "Id": 2,
      "Name": "Constructor Entrance",
      "MachineType": 0,
      "Integrity": 0.9
    },
    {
      "Id": 3,
      "Name": "Constructor Middle",
      "MachineType": 0,
      "Integrity": 0.6
    },
    {
      "Id": 4,
      "Name": "Deconstructor Middle",
      "MachineType": 1,
      "Integrity": 0.8
    }
  ]
}
```

You need a matching C# class to be able to get an C# object out of this JSON text file. The nesting and hirarchy, Naming and value types are important to be able to read and deserialize the file. Matching classes and enum would be the following:

```csharp
using System.Text.Json.Serialization;

namespace DeSerializeJson.Objects
{
    public enum MachineType
    {
        Constructor,
        Deconstructor,
        Welding,
        Laser
    }

    public class Machines
    {
        // Use different Property name than name in the JSON file
        [JsonPropertyName("Machines")] 
        public List<Machine> MachineList { get; set; }
    }

    public class Machine
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public MachineType MachineType { get; set; }
        public float Integrity { get; set; }
    }
}
```

With the `[JsonPropertyName("Machines")]` you can use different names as well and give the JSON serializer a hint, what to match to this property. With this you get more flexability in naming the Properties or the JSON names.

And now take a look on how to use all the above:

```csharp
using DeSerializeJson.Objects;
using System.Text.Json;
using System.Text.Json.Serialization;

Console.WriteLine("Listing all the machines from JSON...");

string machinesJson = File.ReadAllText(@"json\Machines.json");
var machines = JsonSerializer.Deserialize<Machines>(machinesJson).MachineList;

foreach (var machine in machines)
{
    Console.WriteLine("################################");
    Console.WriteLine($"Id: {machine.Id}");
    Console.WriteLine($"Name: {machine.Name}");
    Console.WriteLine($"Type: {machine.MachineType}");
    Console.WriteLine($"Integrity: {machine.Integrity}");
    Console.WriteLine();
}
```

First we read the JSON file as text, then we call the deserializer to get an object out of it: in our case its a list of machines with the read values.

[![deserialization example](/assets/images/articles/json-de-serializing/deserialize-example.png)](/assets/images/articles/json-de-serializing/deserialize-example.png)

As you can see, the values of the JSON file are read per nested JSON object of the "machines" list. The enum types are stored as their identifier (int) instead of their name (string). This can be changed with a litte addition: options for the `JsonSerializer` class call. Add the following options variable and you can store the types as readable strings in the JSON file:

```csharp
// options for converting ENUM Names instead of keys
var options = new JsonSerializerOptions
{
    Converters = { new JsonStringEnumConverter(JsonNamingPolicy.CamelCase) }
};
```

[![deserialization example with enum strings](/assets/images/articles/json-de-serializing/deserialize-example-enum-strings.png)](/assets/images/articles/json-de-serializing/deserialize-example-enum-strings.png)


### Serialization Example

Serializing is a little bit easier, bacause you can code first and the JSON file gets written on how you design your classes.

The classes and enum to generate objects and later the json file:

```csharp
using System.Text.Json.Serialization;

namespace DeSerializeJson.Objects
{
    public enum FactoryType
    {
        ConstructionHall,
        DeconstructionHall,
        SmelterHall,
        LaserStation
    }

    public class Factories
    {
        [JsonPropertyName("Factories")]
        public List<Factory> FactoryList { get; set; }
    }

    public class Factory
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public FactoryType FactoryType { get; set; }
        public float MachineCount { get; set; }
    }
}
```

The usage of these classes:

```csharp
Factories factories = new Factories();
factories.FactoryList = new List<Factory>() {
    new Factory() {
        Id = 1,
        Name = "Hall 01",
        FactoryType = FactoryType.ConstructionHall,
        MachineCount = 1},
    new Factory() {
        Id = 2,
        Name = "Hall 02",
        FactoryType = FactoryType.DeconstructionHall,
        MachineCount = 2},
    new Factory() {
        Id = 3,
        Name = "Hall 03",
        FactoryType = FactoryType.LaserStation,
        MachineCount = 5}
};

string factoriesJson = JsonSerializer.Serialize<Factories>(factories, options);
File.WriteAllText(@"json\Factories.json", factoriesJson);
```

This outputs a file in the given path:

```json
{
    "Factories": [
        {
            "Id": 1,
            "Name": "Hall 01",
            "FactoryType": "constructionHall",
            "MachineCount": 1
        },
        {
            "Id": 2,
            "Name": "Hall 02",
            "FactoryType": "deconstructionHall",
            "MachineCount": 2
        },
        {
            "Id": 3,
            "Name": "Hall 03",
            "FactoryType": "laserStation",
            "MachineCount": 5
        }
    ]
}
```

Serialization also is able to write the names of enums, or the ids if you prefer that for you usecase.