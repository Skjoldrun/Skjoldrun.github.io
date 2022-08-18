---
layout: page
title: C# - Logging in libraries, NullLogger
parent: C#
---

# Logging in Libraries

Getting diagnostics and information from a running application is often done by logging. Logging frameworks help a lot to quickly implement and set up that logging for a new application. But what do you do with logging in libraries? How to call lag statements within, but be independent from the calling application? What if two calling applications use different logging frameworks? What if the application doesn't have a logging framework configured at all?

Implementing logging in a library should be as generic and independent from the calling application as possible to prevent dependencies and problems. The calling application should configure a logging instance, but the library should be able to even run without a logger from the caller at all. Here are my experiences with logging from libraries cooked down to a list of dos and don'ts:

## NullLogger and ILogger - Make the library independent

You don't want to implement references for a specific logging framework in your library because every user of it must implement the same logging framework and use these specific DLL alongside the application. Use Microsoft's minimalistic Extensions package with a `ILogger` interface and a `NullLogger.Instance` check for dependency injection by using the NuGet packages `Microsoft.Extensions.Logging` and `Microsoft.Extensions.Logging.Abstractions`. Implement the library to expect an already configured logger instance, or none at all.

**Example:**

```csharp
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Logging.Abstractions;

namespace SomeLib
{
    public class SomeClass : ISomeClass
    {
        private readonly ILogger _logger;

        public SomeClass(ILogger<SomeClass> logger = null)
        {
            _logger = logger ?? NullLogger<SomeClass>.Instance;
        }

        public void SomeMethod()
        {
            _logger.LogInformation("Method was called.");
            // Do something ...
        }
    }
}
```

As you can see in the upper example, this class expects to get a logger instance, implementing the `ILogger` interface, but defaults to null. With setting the `_logger` field, we check if this given logger instance is null. If it is not null, we set the given logger instance. If it is null though, we set the `NullLogger.Instance` from `Microsoft.Extensions.Logging.Abstractions`.
With this implementation, we are independent from the caller, because we can handle all logging instances with `ILogger` implementations, but not having a logger instance given as well.
Working with the `NullLogger.Instance` has a big performance advantage, because all coming logger calls don't have to do nullchecks anymore and reduce CPU time.

**Using the class - Example with Serilog logging framework:**

```csharp
using Microsoft.Extensions.Logging;
using Serilog;
using SomeLib;
using System;

namespace NullLogger
{
    internal class Program
    {
        private static void Main(string[] args)
        {
            // Create serilog logger with custom Initializer class for configuration
            var serilogLogger = Initializer.CreateLoggerConfiguration().CreateLogger();
            // Set this logger als the static logger
            Log.Logger = serilogLogger;

            // Create a MS extensions ILogger instance of the serilog logger
            var loggerFactory = (ILoggerFactory)new LoggerFactory();
            loggerFactory.AddSerilog(serilogLogger);

            Log.Information("Application start");

            // Inject the generated MS extensions ILogger instance logger with Type into the expecting method manually
            var someClass = new SomeClass(loggerFactory.CreateLogger<SomeClass>());

            // Call the method
            someClass.SomeMethod();

            // Class from lib wihtout the logger injected
            var someClassWithoutLogger = new SomeClass();

            // Call the method again
            someClassWithoutLogger.SomeMethod();

            // End program and flush logger
            Log.Information("Application stop");
            Log.CloseAndFlush();

            Console.ReadKey();
        }
    }
}
```

The first `someClass.SomeMethod()` call produces logging output on all configured Serilog sinks. The second call doesn't produce any output and skips the logging.


## Log errors when you catch, not when you throw

Most libraries don't actually know the exact context and circumstances of the calling application to run into an error. Therefore, just log some debugging context if necessary but let the application log the exact problem in its catch block.
It is up to you to decide if this error is too deep of a level in the call stack to be logged.

But beware: if you excessively generate error log entries you might miss the important ones!


## Name and use your placeholders

If you use structured logging (like with Serilog) you should always name your placeholders in logging statements. This improves later analyzing the log by far.

```csharp
public void Greet(string username)
{
    _logger.LogDebug("Hi there, {UserName}!", username);    // good

    _logger.LogDebug($"Hi there, {username}!");             // bad

    _logger.LogDebug("Hi there, " + username + "!" );      // worse

}
```

Logging like the first one makes it possible to later analyze the `UserName`, instead of just having a concatenated string.


## Log for debugging, not to make noise

Use logging for development and debugging purposes, not to make noise with logging everything that happens in your library. Use the proper log levels to have a silent library in production to prevent spamming and uncontrolled log growth. The bigger the log, the less you read the important statements.

# Disclaimer

All the above has helped me in a lot of ways. None of this is absolute and just meant to be helpful to an interested reader.