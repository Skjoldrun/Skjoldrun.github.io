---
layout: page
title: VS Code - Syntax Highlighting C# Extension Bug
parent: Other
---

# VS Code - Syntax Highlighting C# Extension Bug

The C# extension for VS Code is a useful tool and brings code references, usage of additional keybindings and more. But a downside is that it bring its own syntax highlighting that overwrites your theme syntax highlighting by default.

[![CSharp Extension](/assets/images/articles/vs-code-syntax-highlighting/csharp-extension.png)](/assets/images/articles/vs-code-syntax-highlighting/csharp-extension.png)


Comparing the original code highlighting to the overwritten one from the extension with the VS Dark Modern Theme for VS Code:

[![VS Code wrong highlighting](/assets/images/articles/vs-code-syntax-highlighting/code-example-wrong-highlighting.png)](/assets/images/articles/vs-code-syntax-highlighting/code-example-wrong-highlighting.png)


And now an example with correct highlighting:

[![VS Code correct highlighting](/assets/images/articles/vs-code-syntax-highlighting/code-example-correct-highlighting.png)](/assets/images/articles/vs-code-syntax-highlighting/code-example-correct-highlighting.png)


The correct highlighting has much more variety and contrast. It helps to scan and read the code much better and the original code highlighting of the theme is fully used. 

You have to change two settings in your VS Code to get the original highlighting with the active C# extension. They can be found in the JSON file with: 

```json
"csharp.semanticHighlighting.enabled": false,
"editor.semanticHighlighting.enabled": false,
```

In the GUI you can find both settings with searching for `semanticHighlighting`:

[![VS Code settings](/assets/images/articles/vs-code-syntax-highlighting/vs-code-settings.png)](/assets/images/articles/vs-code-syntax-highlighting/vs-code-settings.png)

