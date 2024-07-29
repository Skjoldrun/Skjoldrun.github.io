---
layout: page
title: C# - catch multiple exception types
parent: C#
---

# Catch multiple exception types

With the Try-Catch block you can catch multiple exception types with the same block. This is useful for handling different exception types differently.

If you want to catch SqlExceptions from `System.Data.SqlClient` and handle these for their own, but still want to catch all other Exceptions as well, you can implement something like this:

```csharp
catch (SqlException ex)
{
    // SqlException ex.Number == -2 for catching timeout exceptions
    // see https://stackoverflow.com/questions/29664/how-to-catch-sqlserver-timeout-exceptions
    if (ex.Number == -2)
    {
        // Do something in case of timeouts ...
    }
}
catch (Exception ex)
{
    // Do something in every other exception case ...
}
```

The more specific upper catch is executed if there is such an exception and the lower catch only get executed if there is no specific exception catch before.


## Another example

Here is another example to simply test the behavior:

```csharp
public static async Task Main(string[] args)
{
    try
    {
        Console.WriteLine("Trowing an Exception ...");
        throw new SomeException("Some text");
    }
    catch (SomeException someEx)
    {
        Console.WriteLine($"Catching someEx: {someEx.Message}");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Catching ex: {ex.Message}");
    }
}

public class SomeException : Exception
{
    public SomeException(string message) : base(message)
    {
    }
}
```
