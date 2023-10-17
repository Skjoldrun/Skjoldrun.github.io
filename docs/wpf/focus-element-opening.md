---
layout: page
title: WPF - focus element on opening
parent: WPF
---

# Focus element on opening

To focus a TextBox on opening of the view, you have to work with the FocusManager:

[![view](/assets/images/articles/focus-on-opening/View.png)](/assets/images/articles/focus-on-opening/View.png)

```csharp
<Grid FocusManager.FocusedElement="{Binding ElementName=txtConfig}">
    <TextBox x:Name="txtConfig" />
</Grid>
```

With this linking of the FocusManager to the TextBox Name, the cursor and focus gets set on the TextBox element on the opened View.