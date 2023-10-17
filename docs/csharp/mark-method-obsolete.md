---
layout: page
title: C# - Mark a method as obsolete
parent: C#
---

# Mark a method as obsolete

If you have legacy code, or you have some other old code you want to rewrite, but don't have the possibility to erase the old methods yet, you could mark them as obsolete methods.

[![Stackoverflow answer](/assets/images/articles/mark-method-obsolete/stackoverflow-answer.png)](/assets/images/articles/mark-method-obsolete/stackoverflow-answer.png)

**Source:** *[stackoverflow](https://stackoverflow.com/questions/1759352/how-to-mark-a-method-as-obsolete-or-deprecated)*

```csharp
[Obsolete("Method is deprecated, please use <Namespace>.<Methodname> instead.")]
```
