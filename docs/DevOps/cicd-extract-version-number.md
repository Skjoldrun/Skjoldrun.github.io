---
layout: page
title: DevOps - Extract version number
parent: DevOps
---

# DevOps - Extract version number

The following PowerShell script can be used as task to extract the version number (file version) from an assembly like a exe or a dll:

```shell
$searchPath = "$(Build.ArtifactStagingDirectory)"
$searchExeName = "$(Solution).exe"

Get-ChildItem -Path $searchPath -Filter $searchExeName -Recurse |
    ForEach-Object {
        try {
            $_ | Add-Member NoteProperty FileVersion ($_.VersionInfo.FileVersion)
        } catch {}
        $_
    } |
    Select-Object -ExpandProperty FileVersion -OutVariable BuildVersionNumber

Write-Host "Extracted FileVersion Number: $($BuildVersionNumber)"

$BuildVersionNumber | out-file -filepath "$(Build.ArtifactStagingDirectory)/version.marker"
Write-Host "Stored the versionNumber in: $(Build.ArtifactStagingDirectory)/version.marker"
```

I used this script in some of my DevOps CI/CD pipelines to check on this, or to create a folder with it.
The `$(Build.ArtifactStagingDirectory)` is a variable in the DevOps pipe, as well as the `$(Solution)`. 
The last two lines are to pipe the `$BuildVersionNumber` to a file `version.marker` wich gets uploaded as pipeline artifact to be available in the separated release pipe, too.

A alternative to work with the version number is to write it to a pipeline variable:

```shell
Write-Host "##vso[task.setvariable variable=BuildVersionNumber]$BuildVersionNumber"
```

[![ci-variable](/assets/images/articles/cicd-extract-fileversion/ci-variable.png)](/assets/images/articles/cicd-extract-fileversion/ci-variable.png)


***Note:***
This pipeline variable named `$BuildVersionNumber` is only visible in the same pipeline Job. Other jobs can't read the variable value set in another prior executed job within the same release pipe.