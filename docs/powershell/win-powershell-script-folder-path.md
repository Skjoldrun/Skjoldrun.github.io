---
layout: page
title: PowerShell - Script folder path
parent: PowerShell
---

# Find and use the Script folder path

The script folder path can be useful to find files that are alongside the script, e.g. a json file with settings for execution. Here is a comparison of `$PSScriptRoot` vs `$MyInvocation.MyCommand.Definition` in PowerShell:

## $PSScriptRoot
- **Available in:** PowerShell 3.0+
- **Usage:**
```powershell
Write-Host "Script is running from: $PSScriptRoot"
```

## Pros
- Built-in, reliable
- Works in scripts, modules, and dot-sourced files

## Cons
- Not available in PowerShell < 3.0
- Not defined in the interactive shell

---

## $MyInvocation.MyCommand.Definition
- **Available in:** PowerShell 2.0+
- **Usage:**
```powershell
$scriptDir = Split-Path -Path $MyInvocation.MyCommand.Definition -Parent
Write-Host "Script is running from: $scriptDir"
```

## Pros
- Compatible with older PowerShell versions

## Cons
- Verbose, context-sensitive
- May behave unexpectedly in functions or dot-sourcing
