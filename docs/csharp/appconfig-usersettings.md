---
layout: page
title: C# - App.config Usersettings
parent: C#
---

# App.config Usersettings

Usersettings can be used to store user specific settings besides the application settings in app.config. The application settings files usually is not writable to let the user store settings in it, furthermore there is one app.config application settings file and there could be more than one user, who wants to store settings. Therefore, the usersettings file is located in a writable path in the AppData folder of the user profile. This could be `C:\Users\<username>\AppData\Local\<companyname>\<projectname>_Url_<some Hash>/<version>`, while the company name is determined by the project settings in the AssemblyInfo.cs (rather .NET Framework), or the package settings in `<PropertyGroup><Company> ... </Company></PropertyGroup>` (.NET Core / .NET 6 and above).
The hash value is created on start of the application and is dependent of the startup location path of the exe. If you copy the project exe into two different locations, there will be two folders with different hash values in the upper shown path.


## Creation

Since .NET Core and above you have to create the settings for yourself and set them as usersettings to get handled as such.

[![Usersettings](/assets/images/articles/usersettings/settings.png)](/assets/images/articles/usersettings/settings.png)

The settings can be defaulted here and later be overwritten by the user if you give them the possibility, e.g. with a settings GUI.


## Version Upgrades

The user profile located folder has a subfolder with the assembly version. If you roll out a new version of the app a new folder gets created for the new version and the stored values are written with your default values again.
If you want to keep the user settings with the new assembly version, then you have to use a build in upgrade mechanism. This can be best triggered by the following method:

Create a usersetting named 'UpgradeRequired' and set it to true. Then place the following code in you startup logic:

```csharp
if (Properties.Settings.Default.UpgradeRequired)
{
    Properties.Settings.Default.Upgrade();
    Properties.Settings.Default.UpgradeRequired = false;
    Properties.Settings.Default.Save();
}
```

This copies the former usersetting values to the new folder with the new hash and then stores UpgradeRequired with false. So, every time a new version is deployed you automatically get the upgraded usersettings for the new version.