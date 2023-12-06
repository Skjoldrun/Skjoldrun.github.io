---
layout: page
title: PowerShell - Remote Commands
parent: PowerShell
---

# PowerShell - Remote Commands

You can call PowerShell Commands to be executed on a remote machine with the Invoke-Command. 

```shell
$cred = get-credential

Invoke-Command -ComputerName YOUR_SERVER -ScriptBlock {Start-WebAppPool -Name APPPOOL_NAME} -credential $cred

Invoke-Command -ComputerName YOUR_SERVER -ScriptBlock {Stop-WebAppPool -Name APPPOOL_NAME} -credential $cred
```

The `$cred = get-credential` asks the user for some credentials via log-in Window and stores them for later usage in the commands. Just replace the server and the appPool names.


## Prepare session and configs for double hops

If you have to invoke a command and use certain credentials for a further hop, you have to create a PowerShell Session Configuration on the Remote Computer:

`Register-PSSessionConfiguration -Name AdminCredConfig -RunAsCredential 'YOUR_USERNAME' -Force`

This command registers the configuration with the wanted credentials for remotly executed commands in the PS session.
You can now use this config for preparing the session:

```shell
$Server = 'YOUR_SERVER'
$Username = 'DOMAIN\USERNAME'
$Password = 'PASSWORD'  # Better use secrets, or map this from secrets
$pass = ConvertTo-SecureString -AsPlainText $Password -Force
$SecureString = $pass
$Credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $Username,$SecureString 

Invoke-Command -ComputerName $Server -Credential $Credential -ConfigurationName AdminCredConfig -ScriptBlock {
    # Do Something on the remote machine
} 
```

This solves the second hob missing priviledges f the first approach and lets you access the further remote machines, as long as the used credentials have the proper priviledges of course.

### Using arguments in the ScriptBlock

You even can use outer variables as arguments in the inner ScriptBlock with `$Using:<varname>` or `$args[0]` and a listed `-AgrumentList <var1>, <var2>` after the ScriptBlock.