---
layout: page
title: C# - Loading Embedded Resources
parent: C#
---

# Loading Embedded Resources in .NET 

In .NET, it's possible to include external files such as templates, configuration files, or images alongside your project. These files can either be placed on disk and accessed at runtime, or embedded directly into the assembly (DLL or EXE) as embedded resources.

Embedding resources has the advantage that:

* They are packaged within the assembly, so users cannot accidentally delete or modify them.
* Developers don't need to handle separate deployment for these files.

The trade-off is that accessing embedded resources requires the use of Reflection at runtime, as they are not accessible via direct file paths.

**Example: Loading an Embedded HTML Mail Template**
Here’s a method that demonstrates how to load an embedded HTML file based on a given `MailTemplateType`:

```csharp
/// <summary>
/// Loads an HTML mail template from embedded resources.
/// </summary>
/// <param name="mailTemplate">The name of the template to load.</param>
/// <returns>The HTML content of the mail template.</returns>
private string LoadMailTemplate(MailTemplateType mailTemplate)
{
    string templateName = $"{mailTemplate}.html";
    var result = string.Empty;
    var assembly = Assembly.GetExecutingAssembly();
    var resourceName = assembly.GetManifestResourceNames()
        .FirstOrDefault(name => name.EndsWith(templateName, StringComparison.OrdinalIgnoreCase));

    if (resourceName == null)
        throw new FileNotFoundException($"Embedded resource '{templateName}' not found.");

    using var stream = assembly.GetManifestResourceStream(resourceName);
    using var reader = new StreamReader(stream);
    result = reader.ReadToEnd();
    
    return result;
}
```

**Notes:** 
* Make sure the file's Build Action is set to Embedded Resource in your project.
* Use the `GetManifestResourceNames()` method to debug and verify which resources are actually embedded.
