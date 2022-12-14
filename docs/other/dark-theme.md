---
layout: page
title: SSMS - Dark Theme
parent: Other
---

# Intro 

In SQL Server Management Studio (SSMS) from Microsoft are some themes to choose from: the blue, blue with extra contrast and light. If you are familiar with Visual Studio you might already know there is another theme with dark colors there. This dark theme can be activated for SSMS as well!

[![options window](/assets/images/other/ssms/preview.png)](/assets/images/other/ssms/preview.png)

Switching themes can be done in the options window with **Tools / Options / Environment / General / Color Theme**:

[![options window](/assets/images/other/ssms/options.png)](/assets/images/other/ssms/options.png)

This dark theme is not officially supported by MS and is not finished in every aspect. Most coloring is fine, but some parts, e.g. the context menu can be a little bit hard to read:

[![dark context menu](/assets/images/other/ssms/context-menu.png)](/assets/images/other/ssms/context-menu.png)


# How to activate the dark theme

## powershell script

You can activate the theme by running the following script as administrator:

```batch
powershell -Command "(gc 'C:\Program Files (x86)\Microsoft SQL Server Management Studio 18\Common7\IDE\ssms.pkgundef') -replace '\[\`$RootKey\`$\\Themes\\{1ded0138-47ce-435e-84ef-9ec1f439b749}\]', '//[`$RootKey`$\Themes\{1ded0138-47ce-435e-84ef-9ec1f439b749}]' | Out-File 'C:\Program Files (x86)\Microsoft SQL Server Management Studio 18\Common7\IDE\ssms.pkgundef'"
```

The theme should now be available in the options window and you're done!


## by hand

If you want to activate the theme by hand you have to open the SSMS configuration file with an editor with administrator rights.

The configuration (ssms.pkgundef) file is located at the following locations: 

| Version   | Path                                                                         |
| --------- | ---------------------------------------------------------------------------- |
| SSMS 2016 | C:\Program Files (x86)\Microsoft SQL Server\130\Tools\Binn\ManagementStudio  |
| SSMS 17   | C:\Program Files (x86)\Microsoft SQL Server\140\Tools\Binn\ManagementStudio  |
| SSMS 18   | C:\Program Files (x86)\Microsoft SQL Server Management Studio 18\Common7\IDE |

Once the file is open, scroll down to `Remove Dark theme` heading and add `\\` to deactivate the removal:

[![dark context menu](/assets/images/other/ssms/config-remove-dark-theme.png)](/assets/images/other/ssms/config-remove-dark-theme.png)

Save the file and the theme should be available in your SSMS to activate.


# Problem with updates / upgrades

The theme gets dropped after an update / upgrade and you have to repeat the upper steps again. I recommend the powershell script as it is fast and handy.