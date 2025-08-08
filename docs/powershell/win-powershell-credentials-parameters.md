---
layout: page
title: PowerShell - Credentials as Parameters
parent: PowerShell
---

# Passing Credentials as Parameters in PowerShell

## Script: Start-RemoteSession.ps1
```powershell
param (
    [Parameter(Mandatory=$true)]
    [string]$Username,

    [Parameter(Mandatory=$true)]
    [SecureString]$Password,

    [Parameter(Mandatory=$true)]
    [string]$ComputerName
)

$credential = New-Object System.Management.Automation.PSCredential ($Username, $Password)
$session = New-PSSession -ComputerName $ComputerName -Credential $credential
Invoke-Command -Session $session -ScriptBlock { hostname }
Remove-PSSession -Session $session
```

## Usage
```powershell
$pwd = Read-Host -AsSecureString -Prompt "Enter password"
./Start-RemoteSession.ps1 -Username "DOMAIN\\User" -Password $pwd -ComputerName "TargetComputer"
```


## CI Pipeline Usage

If teh password comes from a secure pipeline variable, the password is still a string within the pipeline and cannot be received and implicitly converted to SecureString. So teh param definition has to be String and the password then to be converted to SecureString in the script:

```powershell
# Parameter
[Parameter(Mandatory=$true)]
[SecureString]$Password

#Convert to SecureString
$SecurePassword = ConvertTo-SecureString $env:PASSWORD -AsPlainText -Force
$Credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $Username,$SecurePassword
```