---
layout: page
title: DevOps - CI/CD Pipeline
parent: DevOps
---

# CI/CD Pipeline 
This is an example for having a MS DevOps repo with library and some unit testing project to trigger a DevOps CI/CD (continuous integration / continuous deployment) pipeline.


## CI (continuous integration)
It's the practice of merging all developers' working copies to a shared mainline several times a day.

In our case, the CI pipeline builds and tests the code on commits to ensure that everything is buildable and the tests don't fail. This also ensures that the code is also buildable and runnable not only on the developer's computer, but also in the pipeline used minimal container for the execution.


## CD (continuous deployment)
This would be an automatic process to grab the results from the CI pipeline and deploy them to targets, for example a webserver to host the built web API.


# DevOps

Microsoft DevOps is a cloud hosted platform to save and manage the repositories and to run CI/CD pipelines. This is a service from Microsoft and runs on MS cloud infrastructure, hence not in our internal company network.


## Create a CI pipe

[![DevOps Menu](/assets/images/articles/DevOps/DevOps_menu.png)](/assets/images/articles/DevOps/DevOps_menu.png)

[![DevOps create pipe](/assets/images/articles/DevOps/DevOps_create_pipe.png)](/assets/images/articles/DevOps/DevOps_create_pipe.png)

[![DevOps select pipe repo](/assets/images/articles/DevOps/DevOps_select_pipe_repo.png)](/assets/images/articles/DevOps/DevOps_select_pipe_repo.png)

[![DevOps select pipe repo type](/assets/images/articles/DevOps/DevOps_select_pipe_repo_type.png)](/assets/images/articles/DevOps/DevOps_select_pipe_repo_type.png)


Edit the YAML pipeline configuration and add or change the commands to be run:

```yaml
# Configures the triggers, like branch names
trigger:
  branches:
    include:
      - '*'
    exclude:
      - 'Beta'
      - 'Alpha'
      - 'Alpha*'

# DevOps Azure Container:
pool:
  vmImage: 'windows-latest'

# # OnPremise Build machine:
# pool:
#   name: OnPremise Pipelines
#   demands: 
#     - Agent.Name -equals YOUR_BUILD_SERVER

# Variables for the following tasks
variables:
  - name: SolutionName
    value: YOUR_SOLUTION_NAME
  - name: solution
    value: '**/*.sln'
  - name: buildPlatform
    value: 'Any CPU'
  - name: buildConfiguration
    value: 'Release'

# List of tasks as steps
steps:
# git checkout the repository 
# fetchDepth: 0 makes is a complete copy with all commits
- checkout: self
  submodules: true
  persistCredentials: true
  fetchDepth: 0

- task: NuGetToolInstaller@1
  displayName: 'Install NuGet Tools'
  inputs:
    checkLatest: true

- task: NuGetCommand@2
  displayName: 'Restore NuGet Packages'
  inputs:
    restoreSolution: '$(solution)'

- task: VSBuild@1
  displayName: 'Build Solution'
  inputs:
    solution: '$(solution)'
    platform: '$(buildPlatform)'
    configuration: '$(buildConfiguration)'

# Testing with specific path and <test[s]>.dll name
- task: VSTest@2
  inputs:
    testSelector: 'testAssemblies'
    testAssemblyVer2: |
      **\*\*test.dll
      !**\*TestAdapter.dll
      !**\obj\**
    searchFolder: '$(System.DefaultWorkingDirectory)'
    runInParallel: true
    runTestsInIsolation: true
    codeCoverageEnabled: true
    platform: '$(buildPlatform)'
    configuration: '$(buildConfiguration)'

# Copy files to a staging dir, see article [DevOps - Build Agent Directories] for more details
- task: CopyFiles@2
  displayName: 'Copy build artifacts to staging folder'
  inputs:
    SourceFolder: '$(System.DefaultWorkingDirectory)'
    Contents: '$(SolutionName)*/bin/$(buildConfiguration)/**'
    TargetFolder: '$(Build.ArtifactStagingDirectory)'

# Upload the artifacts to the DevOps Cloud storage
- task: PublishPipelineArtifact@1
  displayName: 'Publish Pipeline Artifacts'
  inputs:
    targetPath: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
    publishLocation: 'pipeline'

# Version.marker for extracting a semantic file version number and upload a text file with it
- powershell: |
    $searchPath = "$(Build.ArtifactStagingDirectory)"
    $searchExeName = "$(SolutionName).exe"

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
  displayName: Extract and store version number in version.marker

- task: PublishPipelineArtifact@1
  displayName: 'Publish Pipeline Artifacts - version marker'
  inputs:
    targetPath: '$(Build.ArtifactStagingDirectory)/version.marker'
    ArtifactName: 'version.marker'
    publishLocation: 'pipeline'
```

The yaml example pipeline has some comments for the single steps to be run.

**Pipeline steps:**

* get a isolated container with specific OS to run the pipeline steps
* get the code [with submodules]
* install nuGet tools in the container
* use nuGet tools to get the configured nuGet packages for the code project
* build the code project 
* *[run the unit tests for the code project]*
* *[copy artifact files for publishing them]*
* *[publish the artifact files for further processing, like CD pipeline]*
* *[extract a file version number and write it to a text file to upload it]*

*Steps in `[]` are rather optional and dependent on the situation.*


The pipeline editor also offers some help to inject some tasks on the right side:

[![DevOps pipeline editor task help](/assets/images/articles/DevOps/DevOps_pipelineEditor_tasks_help.png)](/assets/images/articles/DevOps/DevOps_pipelineEditor_tasks_help.png)

After clicking the `Save and Run` button you can configure what branch the pipe should be initially created in and start it:

[![DevOps pipeline run overview](/assets/images/articles/DevOps/DevOps_running_pipe_overview.png)](/assets/images/articles/DevOps/DevOps_running_pipe_overview.png)

You can check the single steps execution output:

[![DevOps pipeline single steps output](/assets/images/articles/DevOps/DevOps_running_pipe_steps.png)](/assets/images/articles/DevOps/DevOps_running_pipe_steps.png)

The pipeline yaml file will be committed and saved in the DevOps repository:

[![DevOps repo xaml file](/assets/images/articles/DevOps/DevOps_repo_yaml_file.png)](/assets/images/articles/DevOps/DevOps_repo_yaml_file.png)

The pipeline is executed on Microsoft cloud infrastructure and is isolated in the configured container. Dependent on the DevOps configuration of the company it will be run on a Worker at times and report its state after finishing.

The user who has triggered the pipe will get an eMail with a link and state report of the pipe:

[![DevOps pipeline eMail](/assets/images/articles/DevOps/DevOps_pipeline_eMail.png)](/assets/images/articles/DevOps/DevOps_pipeline_eMail.png)
