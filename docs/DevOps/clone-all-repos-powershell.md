---
layout: page
title: DevOps - PS Clone all repos
parent: DevOps
---

# Clone all repos from a DevOps project with powershell

This Powershell script reads settings from its settings.json file and then clones all the repos from a certain DevOps project. The script would also clone all submodules in the repos if they contain any.


## Preparation

You need to get a personal access token from DevOps:

[![personal access token 01](/assets/images/articles/DevOps/DevOps_PersonalizedAccessTokens_01.png)](/assets/images/articles/DevOps/DevOps_PersonalizedAccessTokens_01.png)

Create a new one, give it a name and expiration date. You need the Read permission for Code.

[![personal access token 02](/assets/images/articles/DevOps/DevOps_PersonalizedAccessTokens_02.png)](/assets/images/articles/DevOps/DevOps_PersonalizedAccessTokens_02.png)

Then copy the token code and keep it secret, like your passwords. 
*This code should not be added to the git repo or get published by accident!*

Place the token in the settings.json and set the other settings if needed.


## Script

```PowerShell

$jsonSettings = Get-Content -Path settings.json | ConvertFrom-Json
$organization = $jsonSettings.Organization
$project = $jsonSettings.Project
$patFilePath = $jsonSettings.PatFilePath
$targetPath = $jsonSettings.TargetBasePath + $project
$personalAccessToken = "YOUR_TOKEN_IN_PAT_FILE"

if($PersonalAccessToken -eq "YOUR_TOKEN_IN_PAT_FILE") {
    Write-Host "You have not set you personal Token to access DevOps in the settings.json!" -ForegroundColor Red
}

if($jsonSettings.IsTest){
    $targetPath = "$($targetPath)\Test"
    Write-Host "Testmode active! TargetPath gets set to $($targetPath)."
}

$base64AuthInfo = [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes(":$($personalAccessToken)"))
$headers = @{Authorization=("Basic {0}" -f $base64AuthInfo)}

$result = Invoke-RestMethod -Uri "https://dev.azure.com/$0rganization/$project/_apis/git/repositories?api-version=6.0" -Method Get -Headers $headers

$result.value.name | ForEach-Object {
    if(-not (Test-Path -Path "$targetPath\$_" -PathType Container)) {
            Write-Host "Cloning repo $_ to $targetPath\$_" -ForegroundColor Cyan
        git clone --recurse-submodules -j8 ("https://$organization@dev.azure.com/$organization/$project/_git/" + [uri]::EscapeDataString($_)) $targetPath/$_
    }
    else {
        Write-Host "Skipped $_. Already exists in $targetPath\$_." -ForegroundColor Yellow
    }
     
}
```

## settings.json

```json
{
    "Organization": "YOUR_ORGANIZATION",
    "Project": "YOUR_PROJECT",
    "PatFilePath": "pat.txt",
    "TargetBasePath": "C:\\Solutions\\Development\\",
    "IsTest": false
}
```

**Example output**

The script checks if the target folder for each repo already exists and skips existing ones. The output gets colored with yellow for skipped and cyan for new repos to be cloned:

[![terminal example](/assets/images/articles/DevOps/Terminal_Example.png)](/assets/images/articles/DevOps/Terminal_Example.png)


The `IsTest` setting is to create a subfolder "Test" in your target path to isolate the cloned repos for testing around with the script.