---
layout: page
title: C# - appsettings User Secrets
parent: C#
---

# Appsettings User Secrets

User Secrets provide a way to override `appsettings` values locally during development. They are stored outside of your project directory (typically in the user profile's AppData folder) and are **never committed to source control**, ensuring that sensitive or developer-specific data remains private.  
When configured correctly, secrets will override values from both `appsettings.json` and `appsettings.{Environment}.json`, such as `appsettings.Development.json`.


## `secrets.json` Storage

To use User Secrets, right-click on your project and select **“Manage User Secrets”**:

[![add secrets json csproj](/assets/images/articles/appsettings-user-secrets/add-user-secrets-csproj.png)](/assets/images/articles/appsettings-user-secrets/add-user-secrets-csproj.png)

This creates a `secrets.json` file outside your project and adds a `UserSecretsId` to your `.csproj` file.  
You can now define any configuration values you want to override. Partial overrides are also supported:

[![secrets json directory](/assets/images/articles/appsettings-user-secrets/secrets-json-directory.png)](/assets/images/articles/appsettings-user-secrets/secrets-json-directory.png)


## Loading and Configuring in Code

To load the secrets into your application, extend your configuration builder with a call to `builder.AddUserSecrets<Program>();`. You can also replace `Program` with a dummy class (e.g.,`UserSecretsMarkerClass`) as long as it is part of the main project.
The key point is that the class must be discoverable so Visual Studio can correctly resolve the associated `UserSecretsId`.

**Example Code of my AppsettingsHelper class that gets called in the beginning of the app:**
```csharp
/// <summary>
/// Builds and caches the configuration. Reloads only if explicitly reset.
/// </summary>
public static IConfiguration GetConfiguration()
{
    if (_configuration != null)
        return _configuration;

    var envVarName = GetEnvVarName();
    var environment = string.IsNullOrWhiteSpace(envVarName)
        ? null
        : Environment.GetEnvironmentVariable(envVarName);

    var builder = new ConfigurationBuilder()
        .SetBasePath(Directory.GetCurrentDirectory())
        .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true);

    if (!string.IsNullOrEmpty(environment))
        builder.AddJsonFile($"appsettings.{environment}.json", optional: true, reloadOnChange: true);

    builder.AddUserSecrets<Program>();
    builder.AddEnvironmentVariables();
    _configuration = builder.Build();

    return _configuration;
}
```

**secrets.json file:**

```json
{
  "Mailing": {
    "UserName": "SomeSecretUser",
    "Password": "SomeSecretPassword"
  }
}
```

These secrets then overwrite the data configured for the service with the call for configuration in Dependency Injection Service extension in the `MailServiceOptions` object.

**MailingServiceCollectionExtensions example**

```csharp
public static class MailingServiceCollectionExtensions
{
    public static IServiceCollection AddMailer(this IServiceCollection services, IConfiguration configuration)
    {
        services.Configure<MailServiceOptions>(configuration.GetSection(ConfigurationSections.Mailer));
        services.AddTransient<IMailService, MailService>();

        return services;
    }
}
```

**Debug preview with overwritten data:**

[![IOptions preview](/assets/images/articles/appsettings-user-secrets/IOptions-preview.png)](/assets/images/articles/appsettings-user-secrets/IOptions-preview.png)