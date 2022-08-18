---
layout: page
title: Test - Code formatting
parent: Tests
---

# Code formatting and highlighting

{% highlight csharp %} 

Mailer.SendMessage(
	subject: "Place subject here",
	message: "Place information for the recipients here ...",
	monospaceMessage: "Place Stacktrace or code here ...",
	notificationType: NotificationType.Information);

{% endhighlight %}


```csharp
Mailer.SendMessage(
	subject: "Place subject here",
	message: "Place information for the recipients here ...",
	monospaceMessage: "Place Stacktrace or code here ...",
	notificationType: NotificationType.Information);
```

{% highlight csharp %} 

```csharp
Mailer.SendMessage(
	subject: "Place subject here",
	message: "Place information for the recipients here ...",
	monospaceMessage: "Place Stacktrace or code here ...",
	notificationType: NotificationType.Information);
```

{% endhighlight %}
