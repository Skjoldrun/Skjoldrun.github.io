---
layout: page
title: C# - catch sql exceptions
---

# Catch SQL exceptions

With the Try-Catch block you can catch multiple exception types, such as SqlExceptions. If you want to catch SqlExceptions from `System.Data.SqlClient` and handle these for their own, you can implement something like this:

```csharp
catch (SqlException ex)
{
    // SqlException ex.Number == -2 for catching timout exceptions
    // see https://stackoverflow.com/questions/29664/how-to-catch-sqlserver-timeout-exceptions
    if (ex.Number == -2)
    {
        // Do something in case of timeouts ...
    }
    // SqlException ex.Number == 2601 || ex.Number == 2627 for catching duplicate key exceptions on isert and on Primary Key contraint
    // See https://stackoverflow.com/questions/6120422/catching-specific-exception
    // Find SQLException Number with "SELECT * FROM sys.messages WHERE text like '%duplicate key%'"
    else if (ex.Number == 2601 || ex.Number == 2627)
    {
        // Do something in case of duplicate kex exceptions ...
    }
    else
    {
        // Do something in case of all other SqlExceptions ...
    }
}
```

To get the SqlException Number just query the db with the following statement:

```sql
SELECT [message_id], [text] FROM sys.messages WHERE text LIKE '%duplicate key%'
```

Output:

| message_id | text |
| ---------- | ---- |
| 987 | A duplicate key insert was hit when updating system objects in database '%.*ls'. |
| 1505 | The CREATE UNIQUE INDEX statement terminated because a duplicate key was found for the object name '%.*ls' and the index name '%.*ls'. The duplicate key value is %ls. |
| 2512 | Table error: Object ID %d, index ID %d, partition ID %I64d, alloc unit ID %I64d (type %.*ls). Duplicate keys on page %S_PGID slot %d and page %S_PGID slot %d. |
| 2601 | Cannot insert duplicate key row in object '%.*ls' with unique index '%.*ls'. The duplicate key value is %ls. |
| 2627 | Violation of %ls constraint '%.*ls'. Cannot insert duplicate key in object '%.*ls'. The duplicate key value is %ls. |
| 3604 | Duplicate key was ignored. |