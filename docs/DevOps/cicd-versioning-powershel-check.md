---
layout: page
title: DevOps - Versioning powershell check
parent: DevOps
---

# Versioning Powershell check

I used the [NerdBank.GitVersion](https://github.com/dotnet/Nerdbank.GitVersioning) tool for a large solution with multiple projects and wanted to check if every assembly has the proper versioning stamp. After right-clicking some of the output files and checking the explorer details, I came to the conclusion to write a script for listing all the wanted infos.

Powershell lets you do this and with some research I came up with the following script:

```shell
Get-ChildItem -Filter *.dll -Recurse |
    ForEach-Object {
        try {
            $_ | Add-Member NoteProperty FileVersion ($_.VersionInfo.FileVersion)
            $_ | Add-Member NoteProperty AssemblyVersion ([Reflection.AssemblyName]::GetAssemblyName($_.FullName).Version)
            $_ | Add-Member NoteProperty ProductName ($_.VersionInfo.ProductName)
            $_ | Add-Member NoteProperty ProductVersion ($_.VersionInfo.ProductVersion)
            $_ | Add-Member NoteProperty LegalCopyright ($_.VersionInfo.LegalCopyright)

        } catch {}
        $_
    } |
    Select-Object Name,FileVersion,AssemblyVersion,ProductName,ProductVersion,LegalCopyright
```

[![Powershell Check](/assets/images/articles/DevOps/DevOps_versioning_powershell_check.png)](/assets/images/articles/DevOps/DevOps_versioning_powershell_check.png)

With `[Reflection.AssemblyName]::GetAssemblyName($_.FullName).` you can get the assemblyINfo properties and with `$_.VersionInfo.` you get infos to the following FileInfo properties:

* OriginalFilename 
* FileDescription  
* ProductName      
* Comments         
* CompanyName      
* FileName         
* FileVersion      
* ProductVersion   
* IsDebug          
* IsPatched        
* IsPreRelease     
* IsPrivateBuild   
* IsSpecialBuild   
* Language         
* LegalCopyright   
* LegalTrademarks  
* PrivateBuild     
* SpecialBuild