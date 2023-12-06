---
layout: page
title: MSI - Analyze MSIs with Orca
parent: Other
---

# Analyse MSIs with Orca

Orca is an Windows tool for inspecting MSI installer packages. It lets you analyze a MSI package as installer for some software, or your own MSI installer and created WiX MSI package. Unfortunately you can't just install Orca, but have to get some other Installer for the Windows Desktop Development Kit, which then contains the Installer for the Orca Package.

Download the SDK here: [Windows SDK](https://developer.microsoft.com/en-us/windows/downloads/windows-sdk/)

[![SDK download](/assets/images/articles/msi-tool-orca/sdk-website-download.png)](/assets/images/articles/msi-tool-orca/sdk-website-download.png)

The downloaded package has the name: `winsdksetup.exe`.

Open this installer and continue until you get to choose the MSI Tools:

[![MSI Tools install](/assets/images/articles/msi-tool-orca/installer-msi-tools-select.png)](/assets/images/articles/msi-tool-orca/installer-msi-tools-select.png)

After installing it, browse to the target directory. THere you'll find the msi install package for Orca at `C:\Program Files (x86)\Windows Kits\10\bin\10.0.22621.0\x86` as file `Orca-x86_en-us.msi`.

After installing it you have the option to open MSI packages with Orca and inspect their values:

[![Orca inspect](/assets/images/articles/msi-tool-orca/orca-inspect.png)](/assets/images/articles/msi-tool-orca/orca-inspect.png)
