---
layout: page
title: DevOps - CI/CD Agent Directories
parent: DevOps
---

# CI/CD Agent Directories 

The Microsoft DevOps Agent uses some directories to work and build in. The on premise build agent uses them as well as the cloud container agent.

There are:

| directory | variable | description |
| --- | --- | --- |
| s | Build.Repository.LocalPath | "sources" - checked out sources and build folder, like your local project folder |
| b | Build.BinariesDirectory | "binaries" - built binaries |
| a | Build.ArtifactStagingDirectory | "artifacts" - staging folder to collect artifacts to be published |

Check the [predefined variables](https://learn.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops&tabs=yaml) for referencing directories and variables in your pipelines.