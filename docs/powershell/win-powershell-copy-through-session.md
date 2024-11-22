---
layout: page
title: PowerShell - Copy file through a session
parent: PowerShell
---

# PowerShell - Copy file through a session

The following script show how to copy file through a PowerShell session.
This can be useful to reduce used network protocols and needed firewall configs.

The command is `Copy-Item -Path $SourcePath -Destination $TargetPath -ToSession $Session -Verbose -UseSSL` and uses encryption and a prior opened session.

```json
{
    "Username": "SERVICE_USER",
    "Password": "PASSWORD",
    "Machine": "REMOTE_MACHINE",
    "SourcePath": "SOURCE_PATH",
    "TargetPath": "TARGET_PATH_ON_REMOTE_MACHINE"
}
```

```shell
# Variables from Settings JSON
$JsonSettings = Get-Content -Path settings.json | ConvertFrom-Json
$Machine = $JsonSettings.Machine
$Username = $JsonSettings.Username
$Password = $JsonSettings.Password
$TargetPath = $JsonSettings.TargetPath
$SourcePath = $JsonSettings.SourcePath

$SecureString = ConvertTo-SecureString -AsPlainText $Password -Force
$Credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $Username,$SecureString 

Write-Host
Write-Host "Copy Files to remote machine ($TargetPath) started: "

# Create Session
Write-Host 
Write-Host "Create PowerShell Session as User $Username to target machine $Machine ..."
$Session = New-PSSession -ComputerName $Machine -Credential $Credential
if ($Session) {
    Write-Host "Powershell session created." -ForegroundColor Green
} else {
    Write-Host "Error while trying to open a session to $Machine. Error: $_" -ForegroundColor Red
    exit 1
}

# Copy Item through session
Write-Host 
Write-Host "Copy files to target machine $Machine ..."
try {
    Copy-Item -Path $SourcePath -Destination $TargetPath -ToSession $Session -Verbose -UseSSL
    Write-Host "Files copied successfully."
} catch {
    Write-Host "Error while copying files through session to $Machine." -ForegroundColor Red 
    Write-Host "Error: $_" -ForegroundColor Red
    if ($Session) {
        Remove-PSSession -Session $Session
        Write-Host "PowerShell session closed." -ForegroundColor Red
    }
    exit 1
}

# close session
if ($Session) {
    Remove-PSSession -Session $Session
    Write-Host "PowerShell session closed." -ForegroundColor Green
}

Write-Host "Task done."
```
