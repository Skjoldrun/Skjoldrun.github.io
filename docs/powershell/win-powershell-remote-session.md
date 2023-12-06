---
layout: page
title: PowerShell - Remote Session
parent: PowerShell
---

# PowerShell - Remote Session

Instead of invoking a command via `Invoke-Command -ComputerName YOUR_SERVER -ScriptBlock {YOUR_COMMANDS} -credential $cred` on a remote computer, it is also possible to enter a PowerShell remote session via shell on this computer and operate as if you are calling the commands directly on the computer.

```shell
$cred = get-credential
$sess = New-PSSession -Credential $cred -ComputerName REMOTE_COMPUTER
Enter-PSSession $sess
```

The `$cred = get-credential` asks the user for some credentials via log-in Window and stores them for later usage in the commands. 

Configure and store the session with `$sess = New-PSSession -Credential $cred -ComputerName REMOTE_COMPUTER` and replace the COmputerName with your remote computer.

Then entewr the session with `Enter-PSSession $sess`.

The prompt of your shell gets updated and you can see the connected host. All the commands will be executed with the given credentials on the remote computer, as if you would type on its shell directly.

Use `Exit-PSSession`to exit the session and return to the origin computer.


# Double Hop remote execution

I had a case where I needed to enter a remote session from our Build server to a production server and execute a conosle application, which fetches files on further remote hosts in a seperate VLAN. That means i had to double hop from my build server to the production server to several hosts. 
This hopping shuald later be executed as DevOps Release Pipe Tasks.

The upper session was not able to execute the tool on the production server with the correct needed credantials and therefore failed with access denied exceptions. I thought the credentials and then used usercontext with the remote PowerShell session would do the trick but you need a further configuration step on the remote session target machine to use specific credentials. 

Create a PowerShell Session Configuration on the Remote Computer:

`Register-PSSessionConfiguration -Name AdminCredConfig -RunAsCredential 'YOUR_USERNAME' -Force` 

This command registers the configuration with the wanted credentials for remotly executed commands in the PS session.
You can now use this config for opening the session:

```shell
Enter-PSSession -ComputerName YOUR_REMOTE_COMPUTER -Credential $Cred -ConfigurationName AdminCredConfig
```

This solves the second hob missing priviledges f the first approach and lets you access the further remote machines, as long as the used credentials have the proper priviledges of course.