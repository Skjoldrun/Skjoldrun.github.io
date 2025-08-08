---
layout: page
title: PowerShell - Credentials from json
parent: PowerShell
---

# Securely Loading Credentials from settings.json in PowerShell

## Example settings.json
```json
{
  "Username": "DOMAIN\\User",
  "Password": "StrongPassword123!"
}
```

## PowerShell Script to Load and Use the Credentials
```powershell
$jsonPath = Join-Path -Path $PSScriptRoot -ChildPath 'settings.json'

$config = Get-Content -Path $jsonPath -Raw | ConvertFrom-Json
$securePassword = ConvertTo-SecureString -String $config.Password -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential ($config.Username, $securePassword)

$session = New-PSSession -ComputerName "TargetComputer" -Credential $credential
Invoke-Command -Session $session -ScriptBlock { hostname }
Remove-PSSession -Session $session
```

> ⚠️ Avoid using plain-text credentials in shared environments.
