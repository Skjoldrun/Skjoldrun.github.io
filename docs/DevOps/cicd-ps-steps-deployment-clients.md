---
layout: page
title: DevOps - PS steps to deploy clients
parent: DevOps
---

# DevOps Release Pipe PS steps

This is an example how you could set up a release pipeline to deploy clients on remote pcs in your company network, using [Nerdbank Git Versioning (NBGV)](https://github.com/dotnet/nbgv), MS DevOps Release Pipes and Powershell. The single steps are displayed as single PowerShell scripts, but are already prepared to utilize DevOps Release pipe variables and can be grouped and organized together as Task Group in DevOps.


## Manually trigger setup

For easier development the scripts are set up to either be executed manually on the Build Server that usually executes the DevOps Agent and gets remote controlled by the DevOps WebUI, so that you can first develop and test the script steps isolated and later implement them as pipeline. Manually triggered scripts can make sure that you test the Network, Firewalls, Privileges and all necessary configuration before executing these via DevOps pipe, which is a little bit slower due to its overhead.

### Setup and settings.json

The settings.json file can be used as replacement for the DevOps pipeline variables.
Place it besides to the rest of the scripts. The scripts then read the file on startup and prepare the needed variables for execution.

```json
{
    "Username": "SERVICE_DEPLOYMENT_USER",
    "Password": "SERVICE_DEPLOYMENT_USER_PASSWORD",
    "Machine": "TARGET_CLIENT_PC",
    "SourcePath": "YOUR_NETWORK_SHARE_FOR_ARTIFACTS_WITH_VERSION",
    "MachineClientInstallationPath": "C:\\Program Files\\YOUR_APPLICATION_NAME.DeploymentTest\\",
    "BackupServer": "YOUR_SERVER_FOR_BACKUPS",
    "BackupExePath": "PATH_TO_YOUR_BACKUP_TOOL",
    "MachineId": 120,
}
```

With `Machine` you can either use the IP or the Hostname of your target client.
The `SourcePath` is the path to your Artifacts which you want to deploy on the target client pcs. We use the `$BuildVersionNumber`already built in with NBGV as semantic version number.
The `MachineClientInstallationPath` is the target installation path for our client application.
The last three variables `BackupServer`, `BackupExePath` and `MachineId` are used for my own backup solution that can identify client pcs by id or type parameter and creates a snapshot of the install folder and its config files on the client, packs them to zips and copies the zip to the Backupserver. This enables me to have a full revert if the deployment was not successful or there is a critical bug in the new deployed version.


```code   
  -----------------    -----------------    -----------------  
 |                 |  |                 |  |                 | 
 | Build server    |  | Backup server   |  | Artifacts share | 
 |                 |  |                 |  |                 | 
  -----------------    -----------------    -----------------   


  -----------------    -----------------    -----------------    -----------------    -----------------  
 |                 |  |                 |  |                 |  |                 |  |                 | 
 | Client PC       |  | Client PC       |  | Client PC       |  | Client PC       |  | Client PC       | 
 |                 |  |                 |  |                 |  |                 |  |                 | 
  -----------------    -----------------    -----------------    -----------------    -----------------  
```

## Template for using PowerShell Sessions

```shell
# Variables from Settings JSON
$JsonSettings = Get-Content -Path settings.json | ConvertFrom-Json
$Username = $JsonSettings.Username
$Password = $JsonSettings.Password
# YOUR NEEDED VARS HERE

# Variables from DevOps Pipeline
# $Username = '$(serviceUser)'
# $Password = '$(servicePassword)'
# YOUR NEEDED VARS HERE

$SecureString = ConvertTo-SecureString -AsPlainText $Password -Force
$Credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $Username,$SecureString 

# Ping target (optional)
Write-Host 
Write-Host "Try to ping remote computer $Machine ..."
$PingResult = Test-Connection -ComputerName $Machine -Count 1 -Quiet
if ($PingResult -eq $false) {
    Write-Host "Could not reach the remote computer $Machine!" -ForegroundColor Red
    exit 1
} else {
    Write-Host "The remote computer $Machine could be reached successfully!" -ForegroundColor Green
}

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

# Create payload script
$ScriptBlock = {
    try {
        # Write your code here
        # This gets executed on later call in the script and runs on the remote machine through the session

    } catch {
        throw $_
    }
}

# execute script through session
try {
    Invoke-Command -Session $Session -ScriptBlock $ScriptBlock -ErrorAction Stop
} catch {
    Write-Host "Error while executing script block through session on $Machine." -ForegroundColor Red 
    Write-Host "Error: $_" -ForegroundColor Red
    if ($Session) {
        Remove-PSSession -Session $Session
        Write-Host "PowerShell session closed." -ForegroundColor Red
    }
    exit 1
}

# close session
Write-Host "Close PowerShell session to target machine $Machine ..."
if ($Session) {
    Remove-PSSession -Session $Session
}
Write-Host "PowerShell session closed." -ForegroundColor Green

Write-Host "Task done."
```

## Step 01 - Check Host Application

The first step checks if the remote machine can be reached by ping.
Then it creates the powershell session and prepares the script block that should run on the remote machine. 
On the remote machine the script then looks for executables in the configured install target folder and checks if there already is an executable. Then it would check if there is a process still running this executable. This would make sure that the process is stopped and there is no file handle left on the target install files.
If the target install folder is not there, then it could be the first deployment and the script informs about that.

These actions are wrapped with try catch and would elevate possible exception from within the session to its outside and we can handle them with maybe stopping the pipe, if needed. 

```Shell
# Variables from Settings JSON
$JsonSettings = Get-Content -Path settings.json | ConvertFrom-Json
$Username = $JsonSettings.Username
$Password = $JsonSettings.Password
$Machine = $JsonSettings.Machine
$MachineClientInstallationPath = $JsonSettings.MachineClientInstallationPath

# Variables from DevOps Pipeline
# $MachineClientInstallationPath = "$(machineClientInstallationPath)"
# $Username = '$(serviceUser)'
# $Password = '$(servicePassword)'

$SecureString = ConvertTo-SecureString -AsPlainText $Password -Force
$Credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $Username,$SecureString 

# Ping target
Write-Host 
Write-Host "Try to ping remote computer $Machine ..."
$PingResult = Test-Connection -ComputerName $Machine -Count 1 -Quiet
if ($PingResult -eq $false) {
    Write-Host "Could not reach the remote computer $Machine!" -ForegroundColor Red
    exit 1
} else {
    Write-Host "The remote computer $Machine could be reached successfully!" -ForegroundColor Green
}

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

# Create payload script
$ScriptBlock = {
    try {
        Write-Host "Try to find Exe file on remote computer $Machine ..."
        $ExeFile = Get-ChildItem -Path $Using:MachineClientInstallationPath -Filter *.exe -ErrorAction SilentlyContinue | Select-Object -First 1
        if ($ExeFile) {
            $ExeFileName = [System.IO.Path]::GetFileNameWithoutExtension($ExeFile.Name)
            Write-Host "Executable file found: $ExeFileName" -ForegroundColor Green
            
            Write-Host
            Write-Host "Check if remote computer $Using:Machine is currently executing $ExeFileName ..."
            if (Get-Process -Name $ExeFileName -ComputerName $Using:Machine -ErrorAction SilentlyContinue) {
                throw "Process '$ExeFileName' is still running on $Using:Machine!"
            } else {
                Write-Host "Process '$ExeFileName' was not found on $Using:Machine." -ForegroundColor Green
            }
        } else {
            Write-Host "No executable files found in $Using:MachineClientInstallationPath on $Using:Machine." -ForegroundColor Yellow
            Write-Host "This can happen for first deployment or with wrong search path." -ForegroundColor Yellow
            Write-Host "Was not able to test if the target application is still running on remote computer." -ForegroundColor Yellow
            Write-Host "Pipeline execution will continue with next steps." -ForegroundColor Yellow
        }
    } catch {
        throw $_
    }
}

# execute script through session
try {
    Invoke-Command -Session $Session -ScriptBlock $ScriptBlock -ErrorAction Stop
} catch {
    Write-Host "Error while executing script block through session on $Machine." -ForegroundColor Red 
    Write-Host "Error: $_" -ForegroundColor Red
    if ($Session) {
        Remove-PSSession -Session $Session
        Write-Host "PowerShell session closed." -ForegroundColor Red
    }
    exit 1
}

# close session
Write-Host "Close PowerShell session to target machine $Machine ..."
if ($Session) {
    Remove-PSSession -Session $Session
}
Write-Host "PowerShell session closed." -ForegroundColor Green

Write-Host "Task done."
```

## Step 02 - Extract Build Number

This step extracts the build number of the artifacts and stores it for later steps. This is only done by the DevOps Release pipeline, that I have setup to get a file called `version.marker` along with the artifacts to be deployed. The `$BuildVersionNumber` is a DevOps release pipe variable that gets updated for later use.

```shell

Write-Host
Write-Host "Try to extract build version number ..."

$SearchPath = "$(System.DefaultWorkingDirectory)/_$(SolutionName)_master/version.marker/version.marker"
$BuildVersionNumber = Get-Content $SearchPath -Raw

Write-Host "Extracted FileVersion Number: $BuildVersionNumber" -ForegroundColor Green
Write-Host "##vso[task.setvariable variable=buildVersionNumber]$BuildVersionNumber"

Write-Host "Task done."
```

## Step 03 - Check Artifacts 

This step is also a DevOps Only step for me, because I prepare the Client Deployment with the additional preparation steps that place the artifacts for deployment on a network share in folders with their build version numbers. To access this I use the DevOps pipe variable `$ArtifactTargetFolderPath`, that holds a path something like this: 

`\\NETWORKSHARE\ProjectName\BuildVersionNumber\*`


```shell

Write-Host
Write-Host "Checking if artifacts folder exists for $(ProjectName) and deployment preparation is ready ..."

if ($BuildVersionNumber -eq "" ) {
    Write-Host "BuildVersionNumber was empty, check if preparation Task was executed before!" -ForegroundColor Red
    exit 1
}

if (Test-Path -Path $(ArtifactTargetFolderPath)) {
    Write-Host "Artifact targets exist: $(ArtifactTargetFolderPath)" -ForegroundColor Green
} else {
    Write-Error "Artifact targets missing: $(ArtifactTargetFolderPath)" -ForegroundColor Red
    exit 1
}

Write-Host "Task done."
```

## Step 04 - Backup

Next step is to make backups of the already existing application and everything that you need to store, like config files that are machine dependent.

I use my own backup tool, like described before. But if you don't have a backup tool, you could script something that creates zip files and collects them through the session (*I do a similar copy files step later for deployment if you need inspiration how to do that*).

```shell
# Variables from Settings JSON
$JsonSettings = Get-Content -Path settings.json | ConvertFrom-Json
$ProdServer = $JsonSettings.ProdServer
$Username = $JsonSettings.Username
$Password = $JsonSettings.Password
$MachineId = $JsonSettings.MachineId
$PlantBackupExePath = $JsonSettings.PlantBackupExePath

# Variables from DevOps
# $MachineId = '$(machineId)'
# $PlantBackupExePath = '$(plantBackupExePath)'
# $ProdServer = '$(prodServer)'
# $Username = '$(serviceUser)'
# $Password = '$(servicePassword)'

$SecureString = ConvertTo-SecureString -AsPlainText $Password -Force
$Credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $Username,$SecureString 

Write-Host
Write-Host "Executing PlantBackup for machineId $MachineId ..."

Invoke-Command -ComputerName $ProdServer -Credential $Credential -ConfigurationName ProdDeployCredConfig -UseSSL -ScriptBlock {
    Set-Location ("D:\" + $Using:PlantBackupExePath)
    .\PlantBackup.exe $Using:MachineId
} 

Write-Host "Task done."
```

## Step 05 - Prepare Client Path

This steps now would create the application target folder if this is the first deployment, or cleans a existing folder to ensure that the deployment is a fresh one with no old files.

```shell
# Variables from Settings JSON
$JsonSettings = Get-Content -Path settings.json | ConvertFrom-Json
$Machine = $JsonSettings.Machine
$Username = $JsonSettings.Username
$Password = $JsonSettings.Password
$MachineClientInstallationPath = $JsonSettings.MachineClientInstallationPath

# Variables from DevOps
# $MachineClientInstallationPath = "$(machineClientInstallationPath)"
# $ProdServer = '$(prodServer)'
# $Username = '$(serviceUser)'
# $Password = '$(servicePassword)'

$SecureString = ConvertTo-SecureString -AsPlainText $Password -Force
$Credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $Username,$SecureString 

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

# Create payload script
$ScriptBlock = {
    try {
        Write-Host
        Write-Host "Check if Application folder on remote machine exists for ..."
        if (!(Test-Path -Path $Using:MachineClientInstallationPath)){
            New-Item -ItemType directory -Path $Using:MachineClientInstallationPath
            Write-Host "Application folder did not exist and was created: $Using:MachineClientInstallationPath ..." -ForegroundColor Yellow
        } else {
            Write-Host "Application folder is ready." -ForegroundColor Green
        }

        Write-Host
        Write-Host "Clean Application folder with deleting files and Subfolders ..."
        Get-ChildItem -Path $Using:MachineClientInstallationPath -Include "*.*" -Recurse | ForEach-Object { $_.Delete() }
        # loop until no empty folder are left
        do {
            $emptyFolders = Get-ChildItem -Path $Using:MachineClientInstallationPath -Recurse | Where-Object { $_.PSISContainer -and @( $_ | Get-ChildItem ).Count -eq 0 }
            $emptyFolders | Remove-Item
        } while ($emptyFolders.Count -gt 0)

        Write-Host "Deleting files and Subfolders done." -ForegroundColor Green
    } catch {
        throw $_
    }
}

# execute script through session
try {
    Invoke-Command -Session $Session -ScriptBlock $ScriptBlock -ErrorAction Stop
} catch {
    Write-Host "Error while executing script block through session on $Machine." -ForegroundColor Red 
    Write-Host "Error: $_" -ForegroundColor Red
    if ($Session) {
        Remove-PSSession -Session $Session
        Write-Host "PowerShell session closed." -ForegroundColor Red
    }
    exit 1
}

# close session
Write-Host "Close PowerShell session to target machine $Machine ..."
if ($Session) {
    Remove-PSSession -Session $Session
}
Write-Host "PowerShell session closed." -ForegroundColor Green

Write-Host "Task done."
```

## Step 06 - Deploy the artifacts

This is now the final step which deploys the artifacts by copying the files to the remote client machine. It tunnels the copy process through the PowerShell session to prevent a SMB direct access, which makes it easier to maintain and reduces open ports and maybe firewall rules, if you have a segmented network, like the one I have to deal with.

```shell
# Variables from Settings JSON
$JsonSettings = Get-Content -Path settings.json | ConvertFrom-Json
$Machine = $JsonSettings.Machine
$Username = $JsonSettings.Username
$Password = $JsonSettings.Password
$MachineClientInstallationPath = $JsonSettings.MachineClientInstallationPath
$SourcePath = $JsonSettings.SourcePath

# Variables from DevOps
# $SourcePath = "$(artifactTargetFolderPath)" + "\*"
# $ProdServer = '$(prodServer)'
# $Username = '$(serviceUser)'
# $Password = '$(servicePassword)'

$SecureString = ConvertTo-SecureString -AsPlainText $Password -Force
$Credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $Username,$SecureString 

Write-Host
Write-Host "Deploy Artifacts to remote machine ($MachineClientInstallationPath) ..."

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
Write-Host "Deploy Application files to target machine $Machine ..."
try {
    Copy-Item -Path $SourcePath -Destination $MachineClientInstallationPath -ToSession $Session -Verbose
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

# Conclusion

THese steps are nice to maintain and make setting up the pipeline to be a generic collection of PowerShell script steps. These can be organized in a DevOps Task Group and then be reused for each client application that you want to deploy. In my case, this is used to deploy machine control applications for production machines, like mixers and fillers, but also scales and others. All use the same setup und only differ through the variable contents.