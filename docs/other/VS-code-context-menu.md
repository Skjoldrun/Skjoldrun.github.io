---
layout: page
title: VS Code - Open with Context Menu
parent: Other
---

# VS Code - Open with Context Menu

If yoiu have forgotten to set this at install time, or your copmany admins didn't do this for you at deployment, you can activate the context menu entries for opening a workspace and file with VS Code even after installation.

You either reinstall VS Code again with set parameters, or you execute the following Registry Editor file.

## Reinstall

[![install parameters](/assets/images/articles/vs-code-context-menu/vs-code-install.png)](/assets/images/articles/vs-code-context-menu/vs-code-install.png)


## Registry Editor File

Create a new file, e.g. `vsCodeOpenFolder.reg` and paste the following content:

```batch
Windows Registry Editor Version 5.00

; Open files
[HKEY_CLASSES_ROOT\*\shell\Open with VS Code]
@="Edit with VS Code"
"Icon"="C:\\Program Files\\Microsoft VS Code\\Code.exe,0"

[HKEY_CLASSES_ROOT\*\shell\Open with VS Code\command]
@="\"C:\\Program Files\\Microsoft VS Code\\Code.exe\" \"%1\""

; This will make it appear when you right click ON a folder
; The "Icon" line can be removed if you don't want the icon to appear

[HKEY_CLASSES_ROOT\Directory\shell\vscode]
@="Open Folder as VS Code Project"
"Icon"="\"C:\\Program Files\\Microsoft VS Code\\Code.exe\",0"

[HKEY_CLASSES_ROOT\Directory\shell\vscode\command]
@="\"C:\\Program Files\\Microsoft VS Code\\Code.exe\" \"%1\""
; This will make it appear when you right click INSIDE a folder
; The "Icon" line can be removed if you don't want the icon to appear

[HKEY_CLASSES_ROOT\Directory\Background\shell\vscode]
@="Open Folder as VS Code Project"
"Icon"="\"C:\\Program Files\\Microsoft VS Code\\Code.exe\",0"

[HKEY_CLASSES_ROOT\Directory\Background\shell\vscode\command]
@="\"C:\\Program Files\\Microsoft VS Code\\Code.exe\" \"%V\""
```

This file uses path for the default installation of VS Code, adjust these if you have installed this elswhere.