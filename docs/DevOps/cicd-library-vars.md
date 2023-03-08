---
layout: page
title: DevOps - Library variables
parent: DevOps
---

# Library Variables

DevOps can have library or group variables, which are available to multiple pipelines. 

## DevOps

[![DevOps Library Vars](/assets/images/other/DevOps/DevOps_library_vars.png)](/assets/images/other/DevOps/DevOps_library_vars.png)

Add a group with a name and description, then configure the Pipeline permissions. You can create multiple variables with names and values, even secret ones.


## YAML Pipeline Access

To access the variables in the YAML Pipeline you have to reference the group name of the variables in th evariables section:

```yaml
variables:
  - group: EDU_NBGV_Vars
  - name: solution
    value: '**/*.sln'
  - name: buildPlatform
    value: 'Any CPU'
  - name: buildConfiguration
    value: 'Release'
```


### Reading the variables

Note that other variables have to be written as list items now, because the group has to be referenced as one. After referencing the group you now can access your library defined variables just like the inline defined ones, e.g. with `$(ReleaseVersion)`.


### Writing the variables

There are multiple ways to write values back to the variables. One would be with calling a REST API of DevOps. You can find this approach in articles and StackOverflow posts. But I think this is cumbersome and bloats the pipeline script.

A better way of writing values is by calling the azure tools via Powershell. This can be done with the installation of the azure extensions: `az extension add --name azure-devops`. You need to know your variable group id as reference to write a value, so you have to find it. One way is to list all groups with ids: `az pipelines variable-group list --top 3 --query-order Asc --output table`. Another way is to open the group in your browser and analyze the url:

[![DevOps Library Vars Gruop Id](/assets/images/other/DevOps/DevOps_library_vars_groupId.png)](/assets/images/other/DevOps/DevOps_library_vars_groupId.png)

To check, if this was the right one, you can now list these variables with `az pipelines variable-group variable list --group-id 1`.

Now we can write the call for updating the value of a specific variable of the group with:

```powershell
az pipelines variable-group variable update --group-id 1 --name ReleaseVersion --value "2.3.4.5"
```

See the full pipeline task here:

```yaml
- powershell: |
    # az extension add --name azure-devops
    # az pipelines variable-group list --top 3 --query-order Asc --output table
    az pipelines variable-group variable list --group-id 1
    az pipelines variable-group variable update --group-id 1 --name ReleaseVersion --value "2.3.4.5"
  env:
    AZURE_DEVOPS_EXT_PAT: $(System.AccessToken)
```


### Needed permissions

Writing needs the Build Service user of the project to be set as Administrator of the Library / group variables:

[![DevOps Library Vars Security](/assets/images/other/DevOps/DevOps_library_vars_security.png)](/assets/images/other/DevOps/DevOps_library_vars_security.png)

Now the `AZURE_DEVOPS_EXT_PAT: $(System.AccessToken)` part of the Task can authenticate the pipeline user and write the value.