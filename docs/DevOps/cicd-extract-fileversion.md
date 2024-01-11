---
layout: page
title: DevOps - CI/CD extract FileVersion
parent: DevOps
---

# CI/CD extract FileVersion

The following PowerShell script extracts the FileVersion from an given exe files (or dll):

```shell
$searchPath = "$(System.DefaultWorkingDirectory)/YOUR_PATH"
$searchExeName = "YOUR_EXE_OR_DLL"

Get-ChildItem -Path $searchPath -Filter $searchExeName -Recurse |
    ForEach-Object {
        try {
            $_ | Add-Member NoteProperty FileVersion ($_.VersionInfo.FileVersion)
        } catch {}
        $_
    } |
    Select-Object -ExpandProperty FileVersion -OutVariable BuildVersionNumber

Write-Host "Extracted FileVersion Number: $($BuildVersionNumber)"
Write-Host "##vso[task.setvariable variable=BuildVersionNumber]$BuildVersionNumber"
```

The script writes the extracted `$BuildVersionNumber` into the pipeline variable of the same name:

[![ci-variable](/assets/images/articles/cicd-extract-fileversion/ci-variable.png)](/assets/images/articles/cicd-extract-fileversion/ci-variable.png)