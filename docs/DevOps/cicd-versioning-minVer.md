---
layout: page
title: DevOps - CI/CD Versioning MinVer
parent: DevOps
---

# VersioningMinVerEDU 

## Intro

I've tested the versioning tool [MinVer](https://github.com/adamralph/minver).
MinVer lets you insert a [SemVer](https://semver.org/spec/v2.0.0.html) compatible Version number into your C# application by setting Git Tags in you project git repository. It looks for the latest git tag and builds the version of it. 

Check the output from console when running the code to see how the version strings get produced as file version and product version of the application:

![console output](/assets/images/other/DevOps/DevOps_minVer_Console.png)


## DevOps

Using this tool does not let you automate the versioning in a simple way without setting git tags as developer and syncing them from your local repo to DevOps, or with setting the git tag in DevOps itself. There are ways to set tags in the pipeline, but you have to manipulate the version number for at least major and minor number as developer at some point, before building the artifacts. 

A way could be to set some kind of CI pipe variable by hand and let the pipe write the tag back to the repo and use it for versioning. This needs some privileges for the build agent to write / contribute to the repo:

![allow Contribute](/assets/images/other/DevOps/DevOps_minVer_contribute.png)

With these permissions you could predefine variables for the pipeline and let them be set as git tags from within the pipe before building the solution:

```yaml
# This needs extra privileges for the agent 
- script: |
    git tag $(minVer_major).$(minVer_minor).$(minVer_patch)
    git push --tags
    git tag
  workingDirectory: $(Build.SourcesDirectory)
  displayName: write git tag and version
```

 This would result to update the version after building:

![exe properties](/assets/images/other/DevOps/DevOps_minVer_exeProps.png)

A disadvantage of this is that the developer has to manually set the numbers in the pipeline variables for the build or release pipe to use incremented version numbers. 
This is cumbersome and will be forgotten regularly.

## Conclusion

This is a nice versioning tool for minimalistic versioning of projects. But if you want to automate the process with some centralized pipeline process, it may not be what you look for.