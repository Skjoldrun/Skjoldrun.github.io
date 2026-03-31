---
layout: page
title: C# - AppData folders & ProgramData
parent: C#
---

# AppData Folders & ProgramData

Windows provides several special folders for applications to store data, configuration, and cached content. These folders are separated by scope (per-user vs. machine-wide) and purpose (roaming vs. local). Understanding which folder to use is important for correct application behavior, especially in enterprise environments with roaming profiles.


## Overview

| Folder | Path | Scope | Roaming |
|---|---|---|---|
| AppData\Roaming | `C:\Users\<username>\AppData\Roaming` | Per-user | Yes |
| AppData\Local | `C:\Users\<username>\AppData\Local` | Per-user | No |
| AppData\LocalLow | `C:\Users\<username>\AppData\LocalLow` | Per-user | No |
| ProgramData | `C:\ProgramData` | Machine-wide | No |


## AppData\Roaming

**Path:** `C:\Users\<username>\AppData\Roaming`  
**Environment variable:** `%APPDATA%`

This folder is intended for user-specific application data that should follow the user across machines in a domain environment (roaming profiles). When a user logs into a different machine, the contents of this folder are synchronized.

**Use for:**
- User preferences and settings
- Small configuration files
- Templates or customizations
- Application state that should be available on any machine the user logs into

**Examples:** NuGet configuration (`NuGet.Config`), VS Code settings, User Secrets (`secrets.json`)

**Access in C#:**

```csharp
string roamingPath = Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData);
// e.g. C:\Users\JohnDoe\AppData\Roaming
```

> **Tip:** Keep the data stored here small. Roaming profiles synchronize this folder over the network at logon/logoff. Large files will slow down the login process.


## AppData\Local

**Path:** `C:\Users\<username>\AppData\Local`  
**Environment variable:** `%LOCALAPPDATA%`

This folder is for user-specific data that is tied to the local machine and should **not** roam. Data here is typically larger, machine-specific, or easily re-created.

**Use for:**
- Caches and temporary data
- Downloaded content or offline copies
- Log files
- Machine-specific settings (e.g., window positions on a specific monitor setup)
- Large data that would be impractical to roam

**Examples:** NuGet package cache (`NuGet\v3-cache`), browser caches, application log files

**Access in C#:**

```csharp
string localPath = Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData);
// e.g. C:\Users\JohnDoe\AppData\Local
```


## AppData\LocalLow

**Path:** `C:\Users\<username>\AppData\LocalLow`

This folder serves the same general purpose as `AppData\Local` but is accessible to processes running with a lower integrity level (sandboxed or limited-trust applications). Regular applications rarely need to use this folder.

**Use for:**
- Data written by sandboxed or low-integrity processes (e.g., Internet Explorer protected mode, certain browser plugins)

**Access in C#:**

```csharp
string localLowPath = Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData) 
    + "Low";
// There is no dedicated SpecialFolder enum value for LocalLow.
// Alternatively, use SHGetKnownFolderPath via P/Invoke for a clean solution.
```


## ProgramData

**Path:** `C:\ProgramData`  
**Environment variable:** `%PROGRAMDATA%` or `%ALLUSERSPROFILE%`

This folder is for application data that is shared across all users on the machine. It is **not** user-specific. Applications typically create a subfolder here for their own data.

**Use for:**
- Shared configuration files used by all users
- License files
- Shared databases or data stores
- Service configuration (Windows services often store their config and data here)
- Logs for services or machine-wide applications

**Examples:** Chocolatey packages, Docker configuration, SQL Server data files

**Access in C#:**

```csharp
string programDataPath = Environment.GetFolderPath(Environment.SpecialFolder.CommonApplicationData);
// e.g. C:\ProgramData
```

> **Note:** Writing to `ProgramData` may require elevated permissions depending on the subfolder's ACLs. When your application creates its own subfolder during installation, set appropriate permissions so that the application can read/write without requiring admin rights at runtime.


## Recommended Folder Structure

When storing data in any of these folders, follow the convention of creating a subfolder with your company and application name:

```
<SpecialFolder>\<CompanyName>\<ApplicationName>\
```

For example:

```
C:\Users\JohnDoe\AppData\Roaming\MyCompany\MyApp\settings.json
C:\ProgramData\MyCompany\MyApp\shared-config.json
```

This avoids naming collisions with other applications.


## Quick Decision Guide

| Question | Folder |
|---|---|
| Should it follow the user to other machines? | `AppData\Roaming` |
| Is it a cache or machine-specific? | `AppData\Local` |
| Is it shared across all users on the machine? | `ProgramData` |
| Is it written by a sandboxed/low-trust process? | `AppData\LocalLow` |
