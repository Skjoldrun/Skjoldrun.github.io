---
layout: page
title: C# - Namespace
parent: C#
---

# Namespace 

In my work projects I have built a lot of smaller tools, but also some libraries and whole systems, containing multiple projects within the solution. 
In the beginning of the process, you have to choose a good namespace for structuring the project(s) and bring them together in a useful structure, even if the projects don’t interact with each other directly. But let time go, you could encounter the situation, that projects should communicate and even merge to a bigger solution. Having a good namespace would be beneficial in this situation!

Here are some recommendations for namespace logic:

`CompanyName.TechnologyName[.Feature][.Design]`

From: [https://stackoverflow.com/questions/918894/namespace-naming-conventions](https://stackoverflow.com/questions/918894/namespace-naming-conventions)


Alternative:

`<Company>.(<Product>|<Technology>)[.<Feature>][.<Subnamespace>]`

Example: `Microsoft.WindowsMobile.DirectX`

From: [https://stackoverflow.com/questions/918894/namespace-naming-conventions](https://stackoverflow.com/questions/918894/namespace-naming-conventions)