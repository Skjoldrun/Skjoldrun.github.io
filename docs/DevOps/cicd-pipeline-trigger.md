---
layout: page
title: DevOps - CI/CD Pipeline Trigger
parent: DevOps
---

# CI/CD Pipeline Trigger

The MS DevOps CI pipeline can have several triggers defined on top of the yaml file. It is possible to have different scenarios covered and this article describes some of them.


## Multiple branches, multiple pipelines

Imagine you have a `main` branch for production releases, wich only gets merged into from the beta branch. The `beta` branch has its source code only tested in it's own beta environment with it's Release pipe. The developers work in the `development` branch and subbranches for features.

The branches could look something like this:

```
                   / ---- feature 02 branch -------------X
                  / ---- feature 01 branch ------\
           /---------- development branch -----------------\
    /---------- beta branch -------------------------------------\
---------- main branch ------------------------------------------------------
```

You could create three build pipes for this and have one pipe that only produces artifacts from the `main` branch and is coupled with the production release pipe. THis build pipe produces the client assemblies, the server assemblies, a MSI setup and maybe an Outlook AddIn.

The `beat` branch build pipe only triggers if the `beta` branch receives updates and produces artifacts for the beta environment release pipe, wich is a staging environment to test code update before releasing them to production.

The `development` build pipe should be triggered from all other branches, like the `development` branch itself, but also the `feature 01` branch, or the `feature\somethingSpecial` branch and others.

This can be done by defining includes and excludes for the trigger in the yaml pipeline files.


### Example Dev Pipe

```yaml
trigger:
  branches:
    include:
      - '*'
    exclude:
      - 'main'
      - 'Beta'
      - 'Alpha*'
```

This will be trigger from all branches, except the `main`, the `Beta` and every branch that start with "alpha". The asterisk seems to event make the exclude case-insensitive!


### Example Beta Pipe

```yaml
trigger:
  branches:
    include:
      - 'Beta'
    exclude:
      - 'main'
      - 'Alpha*'
```

The exclude part is not really necessary here. 
But every zombie movie teaches us, better double tap that, to be sure.

[![Double-Tap](/assets/images/articles/cicd-pipeline-trigger/double-tap.gif)](/assets/images/articles/cicd-pipeline-trigger/double-tap.gif)


## Example main Pipe

```yaml
trigger:
 branches:
   include:
     - 'main'
```

The `main` pipe doesn't need double tapping. Were not talking about zombies here, right? 🧟