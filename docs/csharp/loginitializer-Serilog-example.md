---
layout: page
title: C# - LogInitializer Serilog example
parent: C#
---

# LogInitializer Serilog example

This article describes a Serilog LogInitializer class  which can be used to init a logger from appsettings, from inline code and even create a logger for DI injection on the fly. That's for cases when you have to deal with older project that don't use DI yet.

**Example:**

```csharp
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.Logging;
using Serilog;
using System;

namespace LoggingExample.Utilities;

public static class LogInitializer
{

    private const string FileLogFolderName = "Log";
    private const string FileLogName = "Serilog_SelfLog_";

    /// <summary>
    /// Creates the logger with inline settings.
    /// </summary>
    /// <returns>Logger with inline settings</returns>
    public static Serilog.ILogger CreateLogger(bool selfLog = false)
    {
        if (selfLog)
            StartSerilogSelfLog();
        
        return new LoggerConfiguration()
                .Enrich.FromLogContext()
                .WriteTo.Async(a =>
                {
                    a.File($@"C:\temp\{ThisAssembly.AssemblyName}\logs\log.txt", rollingInterval: RollingInterval.Hour);
                })
#if DEBUG
                .WriteTo.Console()
                .WriteTo.Debug()
#endif
                .CreateLogger();
    }

    /// <summary>
    /// Creates the logger with settings from appconfig and enrichments from code.
    /// </summary>
    /// <param name="appConfig">appConfig built from appsettings.json</param>
    /// <returns>Logger with inline and app.config settings</returns>
    public static Serilog.ILogger CreateLogger(IConfiguration appConfig, bool selfLog = false)
    {
        if (selfLog)
            StartSerilogSelfLog();

        return new LoggerConfiguration()
            .ReadFrom.Configuration(appConfig)
            .Enrich.FromLogContext()
            .CreateLogger();
    }

    /// <summary>
    /// Creates a logger of type T for injecting it manually into a constructor.
    /// </summary>
    /// <typeparam name="T">type of class to inject the logger to</typeparam>
    /// <returns>ILogger instance of type T for the class constructor of type T</returns>
    public static ILogger<T> CreateLoggerForInjection<T>()
    {
        // Create a MS extensions ILogger instance of the serilog logger
        var loggerFactory = (ILoggerFactory)new LoggerFactory();
        loggerFactory.AddSerilog(Log.Logger);

        var logger = loggerFactory.CreateLogger<T>();

        return logger;
    }

    /// <summary>
    /// Starts the Serilog self logging for internal log exceptions.
    /// Creates a log file in the LocalAppDataCompanyFolderPath.
    /// Deletes old selflog files to keep the folder clean.
    /// </summary>
    private static void StartSerilogSelfLog()
    {
        var appSettingsPath = AppDataCommonPathProcessor.GetCompanyFolderPath(ThisAssembly.AssemblyName);
        var fileLogFolderPath = Path.Combine(appSettingsPath, FileLogFolderName);
        var fileLogFilePath = Path.Combine(fileLogFolderPath, FileLogName.Replace(FileLogName, $"{FileLogName}{DateTime.Now:yyyyMMdd-hhmmss.fff}.txt"));

        Directory.CreateDirectory(fileLogFolderPath);
        var serilogSelfLogFile = File.CreateText(fileLogFilePath);
        Serilog.Debugging.SelfLog.Enable(TextWriter.Synchronized(serilogSelfLogFile));
        Serilog.Debugging.SelfLog.Enable(msg => Debug.WriteLine(msg));

        ClearSerilogSelfLog(fileLogFolderPath);
    }

    /// <summary>
    /// Clears the old selflog files from directory.
    /// </summary>
    /// <param name="fileLogFolderPath">path to be cleared</param>
    /// <param name="ageInDays">number of days to hold old selflog files</param>
    private static void ClearSerilogSelfLog(string fileLogFolderPath, int ageInDays = 5)
    {
        string[] selfLogFiles = Directory.GetFiles(fileLogFolderPath);

        foreach (var file in selfLogFiles)
        {
            FileInfo fi = new FileInfo(file);
            if (fi.LastWriteTime < DateTime.Now.AddDays(-ageInDays) && fi.Name.Contains(FileLogName))
            {
                fi.Delete();
            }
        }
    }
}
```

The LogInitializer class uses another utility class to access a special folder for storing application settings outside the user folder or the application folder:

```csharp
using System;
using System.IO;
using System.Reflection;

namespace LoggingExample.Utilities;

public static class AppDataCommonPathProcessor
{
    private const string CompanyName = "YOUR_COMPANY_NAME";

    /// <summary>
    /// Gets a common path for settings and files for the application in the LocalAppData folder, combined with company name and application name.
    /// Creates the directories of the path unless they don't exist.
    /// </summary>
    /// <param name="applicationName">A subfolder with the given application string gets created if provided.</param>
    /// <returns>Path to appsettings folder</returns>
    public static string GetCompanyFolderPath(string applicationName = "")
    {
        string company = ((AssemblyCompanyAttribute)Attribute.GetCustomAttribute(Assembly.GetEntryAssembly(), typeof(AssemblyCompanyAttribute), false)).Company;
        company = string.IsNullOrEmpty(company) ? CompanyName : company.Replace(' ', '_');

        string companyPath = Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData), company, applicationName);
        Directory.CreateDirectory(companyPath);
        return companyPath;
    }

    /// <summary>
    /// Gets a common path for settings and files for the application in the C:\ProgramData folder, combined with company name and application name.
    /// Creates the directories of the path unless they don't exist.
    /// </summary>
    /// <param name="applicationName">A subfolder with the given application string gets created if provided.</param>
    /// <returns>Path to appsettings folder</returns>
    public static string GetProgramDataCompanyFolderPath(string applicationName = "")
    {
        string company = ((AssemblyCompanyAttribute)Attribute.GetCustomAttribute(Assembly.GetEntryAssembly(), typeof(AssemblyCompanyAttribute), false)).Company;
        company = string.IsNullOrEmpty(company) ? CompanyName : company.Replace(' ', '_');

        string companyPath = Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.CommonApplicationData), company, applicationName);
        Directory.CreateDirectory(companyPath);
        return companyPath;
    }

    /// <summary>
    /// Clears files from a directory where the file name matches a given string phrase.
    /// </summary>
    /// <param name="folderPath">path to be cleared</param>
    /// <param name="match">string to recognize files to be deleted, e.g. ".pdf"</param>
    /// <param name="ageInDays">number of days to hold old files</param>
    public static void ClearFilesFromPath(string folderPath, string match, int ageInDays = 5)
    {
        string[] files = Directory.GetFiles(folderPath);

        foreach (var file in files)
        {
            FileInfo fi = new FileInfo(file);
            if (fi.LastWriteTime < DateTime.Now.AddDays(-ageInDays) && fi.Name.Contains(match))
            {
                fi.Delete();
            }
        }
    }
}
```
