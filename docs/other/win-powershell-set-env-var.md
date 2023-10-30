---
layout: page
title: PowerShell - Windows set Environment Var
parent: Other
---

# PowerShell - Windows set Environment Var

The following command is to set an environment variable on system or machine level on a windows system via PowerShell:

```shell
[System.Environment]::SetEnvironmentVariable('ASPNETCORE_ENVIRONMENT','Test',[System.EnvironmentVariableTarget]::Machine)
```

This sets the variable named `ASPNETCORE_ENVIRONMENT` to the value `Test`. Change the last part to `User` if you only want to set the variable on user level.

Check the Variable with:

```shell
[System.Environment]::GetEnvironmentVariable('ASPNETCORE_ENVIRONMENT')
```
