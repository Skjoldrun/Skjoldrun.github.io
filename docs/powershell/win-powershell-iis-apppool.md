---
layout: page
title: PowerShell - Windows IIS appPools
parent: PowerShell
---

# PowerShell - Windows IIS appPools

The following commands can be used to remotely start and stop the IIS Web AppPools via PowerShell:

```shell
$cred = get-credential

Invoke-Command -ComputerName YOUR_SERVER -ScriptBlock {Start-WebAppPool -Name APPPOOL_NAME} -credential $cred

Invoke-Command -ComputerName YOUR_SERVER -ScriptBlock {Stop-WebAppPool -Name APPPOOL_NAME} -credential $cred
```

The `$cred = get-credential` asks the user for some credentials via log-in Window and stores them for later usage in the commands. Just replace the server and the appPool names.