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

[![DevOps Menu](/assets/images/other/DevOps/DevOps_menu.png)](/assets/images/other/DevOps/DevOps_menu.png)

[![DevOps create pipe](/assets/images/other/DevOps/DevOps_create_pipe.png)](/assets/images/other/DevOps/DevOps_create_pipe.png)

[![DevOps select pipe repo](/assets/images/other/DevOps/DevOps_select_pipe_repo.png)](/assets/images/other/DevOps/DevOps_select_pipe_repo.png)

[![DevOps select pipe repo type](/assets/images/other/DevOps/DevOps_select_pipe_repo_type.png)](/assets/images/other/DevOps/DevOps_select_pipe_repo_type.png)


Edit the YAML pipeline configuration and add or change the commands to be run:

```yaml
# .NET Desktop
# Build and run tests for .NET Desktop or Windows classic desktop solutions.
# Add steps that publish symbols, save build artifacts, and more:
# https://docs.microsoft.com/azure/devops/pipelines/apps/windows/dot-net

# Commenting this results in CI builds of all branches
#trigger:
#- main

# set the OS for the container image
pool:
  vmImage: 'windows-latest'

# define and set some variables
variables:
  solution: '**/*.sln'
  buildPlatform: 'Any CPU'
  buildConfiguration: 'Release'

# setup the steps with units of work
steps:
# This step gets the main repo with its git submodules 
- checkout: self
  submodules: true
  persistCredentials: true

# NuGet set to use specific version
- task: NuGetToolInstaller@1
  inputs:
    checkLatest: true

# NuGet Restore command for all packages
- task: NuGetCommand@2
  inputs:
    restoreSolution: '$(solution)'

# Build the project
- task: VSBuild@1
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

# The Tasks CopyFiles@2 and PublishBuildArtifacts@1 prepare the build artifacts for further processing, like a CD pipeline
#- task: CopyFiles@2
#  inputs:
#    SourceFolder: '$(System.DefaultWorkingDirectory)'
#    Contents: '**\bin\$(buildConfiguration)\**'
#    TargetFolder: '$(Build.ArtifactStagingDirectory)'

#- task: PublishBuildArtifacts@1
#  inputs:
#    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
#    ArtifactName: 'drop'
#    publishLocation: 'Container'

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

*Steps in `[]` are rather optional and dependent on the situation.*


The pipeline editor also offers some help to inject some tasks on the right side:

[![DevOps pipeline editor task help](/assets/images/other/DevOps/DevOps_pipelineEditor_tasks_help.png)](/assets/images/other/DevOps/DevOps_pipelineEditor_tasks_help.png)

After clicking the `Save and Run` button you can configure what branch the pipe should be initially created in and start it:

[![DevOps pipeline run overview](/assets/images/other/DevOps/DevOps_running_pipe_overview.png)](/assets/images/other/DevOps/DevOps_running_pipe_overview.png)

You can check the single steps execution output:

[![DevOps pipeline single steps output](/assets/images/other/DevOps/DevOps_running_pipe_steps.png)](/assets/images/other/DevOps/DevOps_running_pipe_steps.png)

The pipeline yaml file will be committed and saved in the DevOps repository:

[![DevOps repo xaml file](/assets/images/other/DevOps/DevOps_repo_yaml_file.png)](/assets/images/other/DevOps/DevOps_repo_yaml_file.png)

The pipeline is executed on Microsoft cloud infrastructure and is isolated in the configured container. Dependent on the DevOps configuration of the company it will be run on a Worker at times and report its state after finishing.

The user who has triggered the pipe will get an eMail with a link and state report of the pipe:

[![DevOps pipeline eMail](/assets/images/other/DevOps/DevOps_pipeline_eMail.png)](/assets/images/other/DevOps/DevOps_pipeline_eMail.png)

