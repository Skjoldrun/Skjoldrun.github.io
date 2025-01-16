---
layout: page
title: C# - Project PostBuild batch
parent: C#
---

# Project PostBuild batch

The `PostBuild` Event can run additional scripts and steps after the dotnet Build process has run. 

You can configure that with adding the following Entries into the `cspoj` file:

```xml
<Target Name="PostBuild" AfterTargets="PostBuildEvent">
	<Exec Command="call PostBuild.bat $(Projectdir) $(OutDir)" />
</Target>
```

This example would call the `PostBuild.bat` with parameters `$(Projectdir)` and `$(OutDir)`. 

```batch
ECHO ### PostBuildEvent ###
REM ECHO Parameters $(Projectdir) $(OutDir) ...
REM ECHO $(Projectdir) is %1
REM ECHO $(OutDir) is %2

ECHO Copy combit ListAndLabel DLLs and files from the project directory to the output directory ...
XCOPY "%1\..\Combit.ListLabel\" "%1\%2" /Y

ECHO Copy Changes.md into _buildconfig ...
XCOPY "%1\Changes.md" "%1\%2\_buildconfig\" /F /R /Y /I
```

In this case the `changes.md`file and some precompiled dlls would be copied into the output directory. THe console output of that lands in the output window of VS Studio.