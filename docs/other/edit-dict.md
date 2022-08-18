---
layout: page
title: Win - Edit the windows dictionary
parent: Other
---

# Edit the windows dictionary

To edit the Windows dictionary and extend it with custom entries you can open a stored file in your user profile at:

```batch
%appdata%\Microsoft\Spelling\de-DE
```

![appdata folder](/assets/images/other/win/appdata-folder.png)

OPen the file `default.dic` with an editor of your choice. Now you can add custom entries and modify already existing ones. Just don't delete the language id: `#LID 1031` in my case.