---
layout: page
title: DevOps - Get Repos as URL list
parent: DevOps
---

# Get Repos as URL list with powershell

This Powershell script reads settings from its `settings.json` file and the personal access token from the `pat.txt` file and generates a markdown formatted list to your repositories. The list also gets copied to your clipboard.


## Preparation

You need to get a personal access token from DevOps:

[![personal access token 01](/assets/images/articles/DevOps/DevOps_PersonalizedAccessTokens_01.png)](/assets/images/articles/DevOps/DevOps_PersonalizedAccessTokens_01.png)

Create a new one, give it a name and expiration date. You need the Read permission for Code.

[![personal access token 02](/assets/images/articles/DevOps/DevOps_PersonalizedAccessTokens_02.png)](/assets/images/articles/DevOps/DevOps_PersonalizedAccessTokens_02.png)

Then copy the token code and keep it secret, like your passwords. 
*This code should not be added to the git repo or get published by accident!*

Place the token in the `pat.txt` file and create a `.gitignore` file with an entry for the pat file, if needed.


## Script

```powerShell
$jsonSettings = Get-Content -Path settings.json | ConvertFrom-Json
$organization = $jsonSettings.Organization
$project = $jsonSettings.Project
$patFilePath = $jsonSettings.PatFilePath
$personalAccessToken = "YOUR_TOKEN_IN_PAT_FILE"

# Get PersonalAccessToken from PAT file
if (Test-Path $patFilePath) {
    $content = Get-Content $patFilePath
    
    if ($content) {
        $personalAccessToken = $content
        Write-Host "Read PAT '$($content)' from file." -ForegroundColor DarkCyan
    } else {
        Write-Host "PAT file was empty." -ForegroundColor DarkRed
        return
    }
} else {
    Write-Host "No PAT file found." -ForegroundColor DarkRed
    return
}

# Base64 encode the personalAccessToken
$base64AuthInfo = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(":$($personalAccessToken)"))

# Get the list of repositories
$response = Invoke-RestMethod -Uri "https://dev.azure.com/$organization/$project/_apis/git/repositories?api-version=6.0" -Method Get -Headers @{Authorization=("Basic {0}" -f $base64AuthInfo)}

# Output the repositories in a markdown tabular list format
# Prepare the Markdown table
Write-Host "Generating Repository table ..." -ForegroundColor DarkCyan

$markdownTable = @()
$markdownTable += "| Repository |"
$markdownTable += "| ---------- |"
$response.value | ForEach-Object {
    $markdownTable += "| [$($_.name)]($($_.webUrl)) |"
}

# Join the table rows into a single string
$markdownString = $markdownTable -join "`n"

# Copy the Markdown string to the clipboard
$markdownString | Set-Clipboard

Write-Host $markdownString -ForegroundColor DarkGreen
Write-Host ""
Write-Host "The generated table was copied to your Clipboard!" -ForegroundColor DarkCyan
```

## settings.json

```json
{
    "Organization": "YOUR_ORGANIZATION",
    "Project": "YOUR_PROJECT",
    "PatFilePath": "pat.txt"
}
```

The script outputs a markdown formatted list of the repos in the project of your organization and gives supports you with error messages and colored outputs.