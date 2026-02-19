---
layout: page
title: PowerShell - Windows set Environment Var
parent: PowerShell
---

# PowerShell - Windows set Environment Var

The following command is to set an environment variable on system or machine level on a windows system via PowerShell:

```shell
[System.Environment]::SetEnvironmentVariable('SETUP_ENVIRONMENT','DEVELOPMENT',[System.EnvironmentVariableTarget]::Machine)
```

This sets the variable named `SETUP_ENVIRONMENT` to the value `DEVELOPMENT`. Change the last part to `User` if you only want to set the variable on user level.

Check the Variable with:

```shell
System.Environment]::GetEnvironmentVariable('SETUP_ENVIRONMENT', [System.EnvironmentVariableTarget]::Machine)
```

If there is no result then the cache of the terminal will not show it yet. Test closing the terminal and reopen it.