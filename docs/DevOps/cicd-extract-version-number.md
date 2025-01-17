---
layout: page
title: DevOps - Extract version number
parent: DevOps
---

# DevOps - Extract version number

## Single Main Project Solution

The following PowerShell script can be used as task to extract the version number (file version) from an assembly like a exe or a dll. This can be used if the solution creates one single main Application assembly, from wich you want to extract the version:

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


## Multiple Projects Solution

If you have a solution with multiple project in it, that create a application assembly for their own (like multiple different clients, or backend and frontend together), then you can use the following script to loop through the output directories in the CI pipe, that get prepared for uploading to the artifacts pipe storage. While looping through the single output directories, we read and store the containing directories and then look for exe or dll files with the expected name of the project. This only works with projects, that use the assembly name like the project and therefore the directory name.

```yaml
- powershell: |
    Write-Host "Extract and store version number in version.marker for each project ..."
    $binDirectories = Get-ChildItem -Path "$(Build.ArtifactStagingDirectory)" -Recurse -Directory -Filter bin

    foreach ($binDir in $binDirectories) {
        $parentDir = Split-Path -Path $binDir.FullName -Parent
        $projectName = Split-Path -Path $parentDir -Leaf
        Write-Host "Project name: $($projectName)"
        $searchPath = $parentDir
        $searchExePattern = "*$projectName*.exe"
        $searchDllPattern = "*$projectName*.dll"

        Write-Host "Searching for executable or dll ..."
        $file = Get-ChildItem -Path $searchPath -Filter $searchExePattern -Recurse -ErrorAction SilentlyContinue |
                Select-Object -First 1
        if (-not $file) {
            $file = Get-ChildItem -Path $searchPath -Filter $searchDllPattern -Recurse -ErrorAction SilentlyContinue |
                    Select-Object -First 1
        }

        if ($file) {
            try {
                Write-Host "Extracting file version for: $($file.FullName)"
                $fileVersion = $file.VersionInfo.FileVersion
                Write-Host "Extracted FileVersion Number for $($projectName): $($fileVersion)"
                $fileVersion | Out-File -FilePath "$($searchPath)/version.marker"
                Write-Host "Stored the version number in: $($searchPath)/version.marker"
            } catch {
                Write-Error "Failed to extract file version for $($projectName)."
            }
        } else {
            Write-Error "Executable or DLL file not found for $($projectName)."
        }
    }
  displayName: 'Extract version.marker'
  continueOnError: true
```

[![multi-project-drop](/assets/images/articles/cicd-extract-fileversion/multi-project-drop.png)](/assets/images/articles/cicd-extract-fileversion/multi-project-drop.png)