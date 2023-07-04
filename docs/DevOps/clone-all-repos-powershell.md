---
layout: page
title: DevOps - clone all repos powershell
parent: DevOps
---

# Clone all repos from a DevOps project with powershell

This Powershell script reads settings from its settings.json file and then clones all the repos from a certain DevOps project. The script would also clone all submodules in the repos if they contain any.


## Preparation

You need to get a personal access token from DevOps:

[![personal access token 01](/assets/images/other/DevOps/DevOps_PersonalizedAccessTokens_01.png)](/assets/images/other/DevOps/DevOps_PersonalizedAccessTokens_01.png)

Create a new one, give it a name and expiration date. You need the Read permission for Code.

[![personal access token 02](/assets/images/other/DevOps/DevOps_PersonalizedAccessTokens_02.png)](/assets/images/other/DevOps/DevOps_PersonalizedAccessTokens_02.png)

Then copy the token code and keep it secret, like your passwords. 
*This code should not be added to the git repo or get published by accident!*

Place the token in the settings.json and set the other settings if needed.


## Usage

Call the script from a terminal:

```batch
.\CloneAllReposDevOpsEDU.ps1
```

**Example output**

The script checks if the target folder for each repo already exists and skips existing ones. The output gets colored with yellow for skipped and cyan for new repos to be cloned:

[![terminal example](/assets/images/other/DevOps/Terminal_Example.png)](/assets/images/other/DevOps/Terminal_Example.png)


The `IsTest` setting is to create a subfolder "Test" in your target path to isolate the cloned repos for testing around with the script.