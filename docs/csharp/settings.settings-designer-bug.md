---
layout: page
title: C# - Settings.Settings designer file bug
parent: C#
---

# Settings.Settings Designer file bug

[![project explorer](/assets/images/articles/Settings.settings-designer-bug/settings-files.png)](/assets/images/articles/Settings.settings-designer-bug/settings-files.png)

I have encountered old projects to sometimes have a bugged Settings.Settings file wich is not coupled and synchronized with its Settings.Designer.cs file any more. Added new entries are then not synchronized with the designer anymore and you can't use them in your code, because the compiler and IntelliSense don't find the Property anymore. Entering the designer file and adding the needed data by hand is no long term solution, because this file should be generated and synchronized by the IDE.


**I found a fix for that scenario:**

* delete the Settings.Designer.cs file
* exclude the Settings.Settings file from the project
* readd the Settings.settings file to the project
* double click it to open the editor GUI

The IDE should now auto generate the Settings.Designer.cs file again and add the correct entries in the csproj file.