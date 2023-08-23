---
layout: page
title: C# - appsettings overwrite app.config
parent: C#
---

# appsettings overwrite app.config

In .NET core and newer the old app.config as XML file gets replaced by the appsettings.`<environment>`.json file to store settings, like logger configurations, connectionstrings and more. The `<environment>` part can determine for which scenario the file gets loaded. You could write "Production", or "development" files to load different settings for the application. Check the artile [appsettings and environments](/docs/csharp/appsettings-environment.md) for more details.

Changing the codebase from the old .NET Framework to the newer .NET can be tricky sometimes, especially if you have to use really old and really inconvenient tech like table adapters 🤮 ...
Table adapter have a auto generated, hard coded link to a connection string stored in the settings or the app.config. If you have to battle with table adapters in a company wide used old DLL then you have the additional complication, that you can't just overwrite the used connection strings with the app.config entries and XML transforms in a CI/CD pipe. DLLs don't look up their app.configs, but use the compiled settings entries. If you reference the DLL in an application which also uses an app.config, you at least can overwrite the DLL connections there. But what if you have to reference an old DLL from a newer Application, for example a.NET 7.0 app using appsettings.json files?


## appsettings.json to Settings converter

Here is an approach to solve this tricky situation: Read the appsettings.json file at startup and overwrite all settings entries for this runtime. This mimics the old overwriting from the app.config file of the calling application:

```csharp
public class AppSettingsAppConfigConverter : IAppSettingsAppConfigConverter
{
    private readonly IConfiguration _config;
    private readonly ILogger<AppSettingsAppConfigConverter> _logger;

    public AppSettingsAppConfigConverter(IConfiguration config, ILogger<AppSettingsAppConfigConverter> logger = null)
    {
        _config = config;
        _logger = logger ?? NullLogger<AppSettingsAppConfigConverter>.Instance;
    }

    public void OverwriteConfigFromAppSettings()
    {
        try
        {
            foreach (PropertyInfo prop in typeof(Settings).GetProperties())
            {
                string appSettingsConnString = _config.GetConnectionString(prop.Name);
                if (string.IsNullOrWhiteSpace(appSettingsConnString) == false)
                {
                    Settings.Default[prop.Name] = appSettingsConnString;
                    _logger.LogDebug("Change connString {Name} to {New}", prop.Name, appSettingsConnString);
                }
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error while overwriting settings connStrings from AppSettings: {Message}", ex.Message);
            throw;
        }
    }
}
```

This class is placed in the DLL itself and cann access the settings with correct usings. It gets the appsettings read configuration via dependency injection. The overwriting method then checks if the compiled Settings have properties with the similar name to the appsettings.json fed connection strings. If they match the Settings connection String gets overwritten by the new one form the config.


## Testing

I have tested this with appsettings.<environment>.json files in switching environments and with table adapters and it worked. The table adapter then use the new connection strings and can be redirected to a test db or the production db as you would expect.

**The old XML structure:**

```xml
<connectionStrings>
    <add name="YOUR_CONNECTION_STRING_NAME" connectionString="YOUR_CONNECTION_STRING" providerName="System.Data.SqlClient" />
</connectionStrings>
```

**The new appsettings structure:**
```json
{
  "ConnectionStrings": {
    "YOUR_CONNECTION_STRING_NAME": "YOUR_CONNECTION_STRING",
  }
}
```