---
layout: page
title: C# - Secrets with DPAPI (Windows)
parent: C#
---

# Using Windows DPAPI in C# to Secure Secrets — With Optional Entropy and JSON Support

Protecting sensitive data such as API keys, connection strings, or passwords is a core concern in any application. When you're working in a Windows environment, the **Data Protection API (DPAPI)** offers a simple and effective way to encrypt data tied to the current user or machine. In this article, we'll explore how to use DPAPI in C#, add optional entropy for enhanced security, and store/retrieve secrets in JSON format.

## What is DPAPI?

Windows DPAPI provides a built-in encryption service that leverages the user's credentials or machine identity. It abstracts away the complexities of key management, making it ideal for storing configuration secrets securely.

There are two scopes for DPAPI:
- **User scope**: Data is encrypted so only the current user can decrypt it.
- **Machine scope**: Any user on the same machine can decrypt the data.

## DPAPI in C#: Encrypting and Decrypting

Let’s start by creating two utility methods:

```csharp
using System;
using System.Security.Cryptography;
using System.Text;

public static class DpapiUtil
{
    // Encrypts plaintext with optional entropy and user/machine scope
    public static byte[] Protect(string plaintext, byte[] entropy = null, DataProtectionScope scope = DataProtectionScope.CurrentUser)
    {
        byte[] data = Encoding.UTF8.GetBytes(plaintext);
        return ProtectedData.Protect(data, entropy, scope);
    }

    // Decrypts ciphertext with the same entropy and scope
    public static string Unprotect(byte[] encryptedData, byte[] entropy = null, DataProtectionScope scope = DataProtectionScope.CurrentUser)
    {
        byte[] decrypted = ProtectedData.Unprotect(encryptedData, entropy, scope);
        return Encoding.UTF8.GetString(decrypted);
    }
}
```

### Optional Entropy

Adding entropy (aka "additional entropy") strengthens the encryption by introducing extra randomness. It acts like a secondary password and must be the same for both encryption and decryption.

Example entropy:
```csharp
byte[] entropy = Encoding.UTF8.GetBytes("my-super-secret-entropy");
```

## Storing Secrets in JSON

Now, let’s define a model for our secrets and handle reading/writing them to JSON files, with the values encrypted via DPAPI.

```csharp
using System.Collections.Generic;
using System.IO;
using System.Text.Json;

public class SecretStore
{
    public Dictionary<string, string> Secrets { get; set; } = new();

    public void AddSecret(string key, string value, byte[] entropy = null)
    {
        byte[] encrypted = DpapiUtil.Protect(value, entropy);
        Secrets[key] = Convert.ToBase64String(encrypted);
    }

    public string GetSecret(string key, byte[] entropy = null)
    {
        if (!Secrets.TryGetValue(key, out var base64)) return null;
        byte[] encrypted = Convert.FromBase64String(base64);
        return DpapiUtil.Unprotect(encrypted, entropy);
    }

    public void SaveToFile(string path)
    {
        string json = JsonSerializer.Serialize(this, new JsonSerializerOptions { WriteIndented = true });
        File.WriteAllText(path, json);
    }

    public static SecretStore LoadFromFile(string path)
    {
        string json = File.ReadAllText(path);
        return JsonSerializer.Deserialize<SecretStore>(json);
    }
}
```

### Usage Example

```csharp
var entropy = Encoding.UTF8.GetBytes("extra-salt");

// Create new store and add secret
var store = new SecretStore();
store.AddSecret("ApiKey", "my-very-secret-api-key", entropy);
store.SaveToFile("secrets.json");

// Later: load and decrypt
var loadedStore = SecretStore.LoadFromFile("secrets.json");
string apiKey = loadedStore.GetSecret("ApiKey", entropy);

Console.WriteLine($"Decrypted API Key: {apiKey}");
```

## Security Considerations

- **Entropy is optional**, but using it is strongly recommended.
- **Never store entropy in plaintext alongside the encrypted data.**
- **DPAPI is Windows-only** — not portable to Linux or macOS.
- **Use `DataProtectionScope.LocalMachine` only for shared applications**, and only if appropriate permissions and security isolation are in place.

## When to Use DPAPI

DPAPI is a solid choice when:
- You need to store secrets locally and securely.
- You want a zero-config, keyless encryption approach.
- You’re targeting Windows environments exclusively.

For cross-platform solutions, consider alternatives like:
- ASP.NET Core `IDataProtectionProvider`
- Azure Key Vault or AWS KMS
- libsodium or BouncyCastle for manual key-based encryption

## Conclusion

Windows DPAPI in C# offers a streamlined and secure method to encrypt application secrets. With optional entropy and JSON-based storage, you can easily integrate this into any configuration system without managing your own encryption keys.
