---
layout: page
title: C# - lock statement
---

# Lock statement

If you want to access objects from multiple threads you could use a locking mechanism for managing the access and preventing race conditions.

While the lock is set and the first accessing thread enters the block, any further thread is blocked and waits until the lock is released from the first thread.

You can lock code execution with the following code:

```csharp
lock (x)
{
    // Your code...
}
```

Check the following link for more details: 
https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/statements/lock