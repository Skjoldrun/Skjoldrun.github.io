---
layout: page
title: DevOps - Solution Filters (.slnf)
parent: DevOps
---

# Solution Filters (.slnf) in the CI Pipeline

Sometimes a solution contains a project that you **want to keep for local development**,
but that must **not** be part of the CI build. Classic example: a legacy or temporary
project that is only referenced during a migration, lives in a folder that is excluded via
`.gitignore`, and therefore never gets checked out on the build agent.

The moment your pipeline tries to restore the solution, it blows up:

```
error MSB3202: The project file
"...\_Temp_Legacy_Code\LegacyApp\LegacyApp.csproj" was not found.
```

This article explains **why** this happens, why "just don't build it" does not help, and how
a **solution filter file** (`.slnf`) solves it cleanly - both locally and in CI.


## The trap: restore does not care about build configuration

A very common first reaction is:

> "I'll just remove the project from the `Release|Any CPU` build configuration so it doesn't
> get built."

That does **not** work. `dotnet restore` (and `MSBuild /t:Restore`) enumerates **every**
project listed in the solution to build the restore graph - completely **independent** of the
build configuration. So even a project that is "excluded from build" is still read during
restore. If its `.csproj` is physically missing on the agent (because the folder is
gitignored), restore fails with `MSB3202` **before** any build-configuration filtering ever
happens.

```
   Solution on the build agent
   ===========================

   AppSuite.Clients.slnx
        |
        |  restore enumerates ALL projects,
        |  regardless of Debug/Release config
        v
   +----------------------+   +----------------------+   +----------------------+
   |  App.Client.csproj   |   |  App.Core.csproj     |   |  LegacyApp.csproj    |
   |  (checked out)  OK   |   |  (checked out)  OK   |   |  GITIGNORED  ✗ 404   |
   +----------------------+   +----------------------+   +----------------------+
                                                                   |
                                                                   v
                                                        error MSB3202: not found
```

So there are only really three ways out:

1. Make the file **exist** on the agent (un-ignore it) - but then it gets restored/built,
   which is exactly what you wanted to avoid for a legacy/temp project.
2. **Remove** it from the solution - but if you keep merging migration work that re-adds it,
   this fight never ends.
3. Feed restore/build a **filtered view** of the solution that simply does not contain that
   project. That is what a `.slnf` is for.


## What a solution filter (.slnf) is

A `.slnf` is a tiny **JSON** file that sits next to your solution and lists the subset of
projects that should be **loaded / restored / built**. It references the real solution file
and includes only what you want:

```
   AppSuite.Clients.slnx              AppSuite.Clients.slnf
   (full solution)                    (filtered view for CI)
   +---------------------------+      +---------------------------+
   |  App.Client               |      |  App.Client               |
   |  App.Core                 |  ->  |  App.Core                 |
   |  App.Tests                |      |  App.Tests                |
   |  LegacyApp   <-- temp     |      |  (LegacyApp omitted)      |
   +---------------------------+      +---------------------------+
          used locally                       used by the CI pipe
```

Key properties that make it a great fit:

- The **solution file stays untouched**. Locally you still open the full solution with the
  legacy project referenced - nothing breaks for developers.
- The filter is **independent of the solution content**. Even if a later merge re-adds the
  legacy project to the solution, it will **never** appear in the filter.
- Project-to-project references are still followed **transitively**. You only need to list the
  projects you actually care about; their dependencies are pulled in automatically. Since
  nothing references the omitted legacy project, it drops out cleanly.


## The .slnf file format

The format is plain JSON. Paths are **relative to the folder the `.slnf` lives in** and use
backslashes:

```json
{
  "solution": {
    "path": "AppSuite.Clients.slnx",
    "projects": [
      "App.Client\\App.Client.csproj",
      "App.Client.Core\\App.Client.Core.csproj",
      "..\\Core\\App.Core\\App.Core.csproj",
      "..\\Core\\App.Grpc.Core\\App.Grpc.Core.csproj",
      "..\\Core\\App.Client.Tests\\App.Client.Tests.csproj"
    ]
  }
}
```

Note: the omitted legacy project (`..\\Core\\_Temp_Legacy_Code\\LegacyApp\\LegacyApp.csproj`)
is simply **not** in the list. That is the whole trick.

> **Compatibility:** Solution filters work with the modern XML solution format (`.slnx`) as
> well as with the classic `.sln`. Building `.slnx` and `.slnf` from the command line is
> supported since the .NET SDK 9.0.200, and `.slnx` is the default solution format from
> .NET 10 onwards. If you migrate `.sln` -> `.slnx`, point the `"path"` in your filter at the
> new `.slnx`.


## Creating a .slnf

You do not have to hand-write it (although it is easy enough). Common ways:

- **Visual Studio:** right-click the projects you want to *unload*, unload them, then use
  *"Save As Solution Filter"* on the solution node. VS writes the `.slnf` listing the loaded
  projects.
- **CLI:** start from the full solution and remove what you do not want, or just author the
  JSON by hand for full control.

Then verify it statically (no build needed) - valid JSON, all listed projects exist, and the
unwanted one is absent:

```powershell
$slnf = Get-Content .\AppSuite.Clients.slnf -Raw | ConvertFrom-Json
$slnf.solution.projects | ForEach-Object {
    $exists = Test-Path (Join-Path (Get-Location) $_)
    "{0,-6} {1}" -f $(if ($exists) {'OK'} else {'MISS'}), $_
}
```


## Using it in the CI pipeline

The neat part: a `.slnf` can be handed to the exact same tasks you already use for a solution.
Just point your restore and build tasks at the filter instead of the solution.

### Before - restoring the full solution (fails on the missing project)

```yaml
variables:
  - name: client-solution
    value: '**/AppSuite.Clients.slnx'   # includes the gitignored legacy project -> MSB3202
```

### After - restoring the filtered view

```yaml
variables:
  - name: client-solution
    value: '**/AppSuite.Clients.slnf'   # legacy project is filtered out

steps:
# ==== Client Restore ====
- task: DotNetCoreCLI@2
  displayName: 'Restore Client'
  inputs:
    command: restore
    projects: '$(client-solution)'
    feedsToUse: config
    nugetConfigPath: NuGet.config

# ==== Client Build ====
- task: VSBuild@1
  displayName: 'Build Client'
  inputs:
    solution: '$(client-solution)'
    platform: '$(buildPlatform)'
    configuration: '$(buildConfiguration)'
    vsVersion: 'latest'
```

Both `DotNetCoreCLI@2` (restore/build) and `VSBuild@1` accept the `.slnf` transparently, so the
change is usually a **one-line variable swap**.

```
   Pipeline flow
   =============

   checkout ->  restore (*.slnf)  ->  build (*.slnf)  ->  test  ->  publish
                    |                      |
                    |                      +-- legacy project never seen
                    +-- restore graph built WITHOUT the legacy project -> no MSB3202
```


## Trade-offs and gotchas

- **Maintenance point:** the filter is an explicit allow-list. When you add a *new, real*
  project that CI must build, remember to add it to the `.slnf` too - otherwise it silently
  stays out of the CI build.
- **Transitive references only:** a project reached only through a P2P reference of a listed
  project is included automatically. But if you want a project built as a *root*, list it
  explicitly.
- **Not a build-config toggle:** a `.slnf` changes *which projects load*, not *how* they are
  configured. It is the right tool precisely because it operates at the same stage as restore.
- **Keep the solution as the source of truth for humans:** developers still open the full
  `.slnx`. The `.slnf` is a CI/automation concern.


## Conclusion

If your build fails during **restore** with `MSB3202` because the solution references a project
that is not checked out (a gitignored legacy/temp project, a migration leftover, an optional
component), do **not** try to fix it via build configuration - restore ignores that.

Instead, drop a small **solution filter** next to the solution, list everything the CI needs,
leave the unwanted project out, and point your pipeline's solution variable at the `.slnf`. The
full solution stays intact for local work, future merges cannot sneak the project back into the
build, and your pipeline goes green again.
