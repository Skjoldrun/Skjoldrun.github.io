---
layout: page
title: C# - Windows Username
parent: C#
---

# Windows Username

To get information about the current user context, use one of the following:

`Environment.UserName` 

Only the username of the calling user (Run As would get this user)

`System.Security.Principal.WindowsIdentity.GetCurrent().Name`

Gets the Domain and the user `<Domain>\<Username>`

`System.Security.Principal.WindowsIdentity.GetCurrent().User`

Gets the SID of the user e.g. "S-1-5-21-992878714-4041223874-2616370337-1001"

These calls are at least valid for applications like WPF, WinForms and Console Applications.