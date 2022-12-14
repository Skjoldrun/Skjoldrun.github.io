---
layout: page
title: C# - Alertmails & HTML template
parent: C#
---

# Alertmails with Oulook HTML template

I have built an AlertMailing utility for a logging extension once, that can read HTML templates from disc and replaces certain tokens with strings and values to be set from inside the calling code.

**Preview**

[![AlertMail preview](/assets/images/coding/csharp/alertmails-outlook-html-template/preview-info.png)](/assets/images/coding/csharp/alertmails-outlook-html-template/preview-info.png)


The alertMail informs the receiver about the machine it got sent from, the AssemblyName and version, displays a message text and a optional monospaced message block to show even more info, like the callstack.

The mail has prepared links on the bottom, too. These could be set to the company website and DevOps site in the template.

Outlook can handle certain HTML with CSS and this template can also be shown in the light design, or in the dark design for Outllok, as you can see above.


## Call example

```csharp
Mailer.SendMessage(
	subject: "Place subject here",
	message: "Place information for the recipients here ...",
	monospaceMessage: "Place Stacktrace or code here ...",
	notificationType: NotificationType.Information);
```

Call the `SendMessage(...)` method like above and fill in the strings and values you want to display in the mail. Sender and receiver are set by the `config\MailerConfig.json` file in the project folder.


## Config example

```json
{
  "MailHtmlTemplatePath": "MailTemplates\\AlertMail.html",
  "SmtpHost": "mailserver.yourDomain.de",
  "Sender": "AlertMail@yourDomain.de",
  "SpamPreventonSubjectAge": 30,
  "RecipientList": [
    "recipient01@yourDomain.de",
    "recipient02@yourDomain.de"
  ]
}
```


## NotificationType or Level

You can set several notification types or levels by setting the `NotificationType` parameter. The level is then displayed in the mail as string, the subject gets enriched and the color of the background gets changed (blue for Information, green for Healthy, orange for Warning, red for Critical, dark red for failure).


## Mail Template

```html
<html>
	<head>
    	<title>Email Alert</title>
		<!-- ... CSS ... -->
	</head>
	<body style="margin: 0; padding: 0">
		<table style="border: none" cellpadding="0" cellspacing="0" width="100%">
			<tr>
				<td style="padding: 15px 0">
					<table style="border: none;" cellpadding="0" cellspacing="0" width="99%" class="content">
						<tr>
							<td class="general">
								<h1>Email Alert: {{ApplicationName}}</h1>
							</td>
						</tr>
						<tr>
							<td class="{{NotificationType}}">
								<h1>Notification Level: {{NotificationType}}</h1>
								<p>MachineName: <b>{{MachineName}}</b> | AssemblyName: <b>{{AssemblyName}}</b> | AssemblyVersion: <b>{{AssemblyVersion}}</b><br />{{Message}}</p>
								<pre>{{MonospaceMessage}}</pre>
							</td>
						</tr>
						<tr>
							<td class="general">
								<p><a href="https://www.someDomain.de/">Your site</a> | <a href="https://dev.azure.com/">DevOps</a></p>
							</td>
						</tr>
					</table>
				</td>
			</tr>
		</table>
	</body>
</html>
```

The tokens to be replaced are embedded in the HTML with '{{' and '}}'. These get replaced in the Mailer C# method with the parameters of the call or by config and enrichment.

You can find the complete template down below, including the CSS part!
It is set to be fluent and automatically grows with the mail client window in width.


## Mailer class and send methods

The Mailer class holds some fields: the lock object and the dictionary are for storing infos of last sent mails to prevent rapidly sending over the same mail if it gets called in a loop for example. I will show the spam prevention in a later part.

```csharp
public static class Mailer
{
	private static readonly object _lock = new object();
	private static Dictionary<int, DateTime> _lastMailSentDict = new Dictionary<int, DateTime>();
	private static string _applicationName;

	//...
}
```

### Send methods

There are several methods to be called for sending mails. Let me show one of these in detail, the rest is quite similar and mostly differs in the parameters and explicit settings of receiver and sender.

The method show above is the following:

```csharp
public static void SendMessage(
	string subject,
	string message,
	string monospaceMessage = "",
	string applicationName = "",
	NotificationType notificationType = NotificationType.Information)
{
	MailerConfig mailerConfig = GetMailerConfig();

	SendMessageFromTo(sender: mailerConfig.Sender,
		recipients: mailerConfig.RecipientList,
		subject: subject,
		message: message,
		monospaceMessage: monospaceMessage,
		applicationName: applicationName,
		notificationType: notificationType);
}

public static void SendMessageFromTo(
	string sender,
	List<string> recipients,
	string subject,
	string message,
	string monospaceMessage = "",
	string applicationName = "",
	NotificationType notificationType = NotificationType.Information)
{
	MailerConfig mailerConfig = GetMailerConfig();

	if (string.IsNullOrEmpty(applicationName))
	{
		_applicationName = AssemblyInfoProcessor.GetEntryAssemblyName();
	}

	// Prevent spam if the mail was sent within the amount of minutes before.
	// If value was not set in the mailerconfig file all mails get sent.
	if (CheckIfSpam(subject, mailerConfig.SpamPreventionSubjectAge))
	{
		return;
	}

	string mailTemplate = File.ReadAllText(mailerConfig.MailHtmlTemplatePath);
	MailMessage mailMessage = new MailMessage();
	SmtpClient smtpClient = new SmtpClient();

	message = FixNewLineFormattingForPTag(message);
	mailMessage.Subject = $"{_applicationName} - {notificationType}: {subject}";
	mailMessage.From = new MailAddress(sender);
	foreach (var recipient in recipients)
	{
		mailMessage.To.Add(recipient);
	}
	mailMessage.IsBodyHtml = true;
	mailMessage.Body = mailTemplate
		.Replace("{{NotificationType}}", notificationType.ToString())
		.Replace("{{MachineName}}", Environment.MachineName)
		.Replace("{{AssemblyName}}", AssemblyInfoProcessor.GetEntryAssemblyName())
		.Replace("{{AssemblyVersion}}", AssemblyInfoProcessor.GetEntryAssemblyVersion())
		.Replace("{{ApplicationName}}", _applicationName)
		.Replace("{{Message}}", message)
		.Replace("{{MonospaceMessage}}", monospaceMessage);
	mailMessage.BodyEncoding = Encoding.UTF8;
	mailMessage.SubjectEncoding = Encoding.UTF8;
	smtpClient.Host = mailerConfig.SmtpHost;

	TrySendMessage(smtpClient, mailMessage, message, monospaceMessage);
}

private static MailerConfig GetMailerConfig()
{
	Directory.SetCurrentDirectory(AppDomain.CurrentDomain.BaseDirectory);
	string mailerConfigJson = File.ReadAllText(@"Config\MailerConfig.json");
	MailerConfig mailerConfig = JsonSerializer.Deserialize<MailerConfig>(mailerConfigJson);
	return mailerConfig;
}
```

As you can see, the Tokens get replaced at building the `mailMessage.Body`. Calling this method will enrich the subject and body with muliple informations which come in handy for analysing a certain alert or mailed exception.
I wanted to have the parameters to be optional to let the user not have to worry about them, as long as he/she doesn't needs them.

The other Methods then differ in not using the HTML template and only send plain text messages to keep the mail as light as possible.

The method `TrySendMessage(...)` tries to send the mail, but calls a utility class to write the message to the local windows eventLog if sending it as a mail fails.


## Spam prevention

As mentioned above, I have built a little optional spam prevention to prevent sending the same mail (e.g. out of a loop) over and over again. The sent mails get stored by their subject as hash and timestamp. These go into an in-memory dictionary to be compared, or in a written json file on disc to even store sent mail information for console applications in a scheduler. You can overwrite the default 30 min age to prevent sending the same mail again by call or by config.

```csharp
public static bool CheckIfSpam(string subject, int subjectAge = 30, bool spamPreventionStored = false)
        {
            bool isSpam = false;

            // if it is lower or equal to 0, spam prevention is disabled and everything gets sent.
            if (subjectAge <= 0)
            {
                return isSpam;
            }

            if (spamPreventionStored)
            {
                ReadStoredLastMails();
            }

            var subjectHash = subject.GetHashCode();

            if (_lastMailSentDict.TryGetValue(subjectHash, out DateTime lastValue))
            {
                if (lastValue.AddMinutes(subjectAge) < DateTime.Now)
                {
                    lock (_lock)
                    {
                        _lastMailSentDict.Remove(subjectHash);
                        _lastMailSentDict.Add(subjectHash, DateTime.Now);
                    }
                    isSpam = false;
                }
                else
                {
                    lock (_lock)
                    {
                        _lastMailSentDict.Remove(subjectHash);
                        _lastMailSentDict.Add(subjectHash, DateTime.Now);
                    }

                    isSpam = true;
                }
            }
            else
            {
                lock (_lock)
                {
                    _lastMailSentDict.Add(subjectHash, DateTime.Now);
                }

                isSpam = false;
            }

            if (spamPreventionStored)
            {
                WriteStoredLastMails();
            }

            return isSpam;
        }
```


# Complete overview of all used classes and code

Here are the complete classes of the above shown snippets. Feel free to use and tweak them. I have not done a repository and library with these yet, they are embedded in some bigger lib until I build a dedicated lib for this purpose only.


## Class to read config json

```csharp
using System.Collections.Generic;

namespace Production.Mailing
{
    public class MailerConfig
    {
        public string MailHtmlTemplatePath { get; set; }
        public string SmtpHost { get; set; }
        public string Sender { get; set; }
        public int SpamPreventionSubjectAge { get; set; }
        public bool SpamPreventionStored { get; set; }
        public List<string> RecipientList { get; set; }
    }
}
```


## Complete Mailer class  

```csharp
using Production.Utilities;
using System;
using System.Collections.Generic;
using System.IO;
using System.Net.Mail;
using System.Text;
using System.Text.Json;

namespace Production.Mailing
{
    public enum NotificationType
    {
        Failure,
        Critical,
        Warning,
        Healthy,
        Information
    }

    public static class Mailer
    {
        private static readonly object _lock = new object();
        private static Dictionary<int, DateTime> _lastMailSentDict = new Dictionary<int, DateTime>();
        private static string _applicationName;

        /// <summary>
        /// Sends a message to a configured smtp host with a given HTML template and replaces placeholders with the given parameters.
        /// Uses a HTML template and replaces placeholders with the given parameters from mailerconfig.
        /// </summary>
        /// <param name="subject">subject for the mail</param>
        /// <param name="message">normally formatted message</param>
        /// <param name="monospaceMessage">monospace formatted message, eg. callstack, or code</param>
        /// <param name="applicationName">name of the calling application</param>
        /// <param name="notificationType">type for formatting the mail. possible types are: Failure, Critical, Warning, Healthy, Information</param>
        public static void SendMessage(
            string subject,
            string message,
            string monospaceMessage = "",
            string applicationName = "",
            NotificationType notificationType = NotificationType.Information)
        {
            MailerConfig mailerConfig = GetMailerConfig();

            SendMessageFromTo(sender: mailerConfig.Sender,
                recipients: mailerConfig.RecipientList,
                subject: subject,
                message: message,
                monospaceMessage: monospaceMessage,
                applicationName: applicationName,
                notificationType: notificationType);
        }

        /// <summary>
        /// Sends a message to a given recipients list from a given sender and configured smtp host.
        /// Uses a HTML template and replaces placeholders with the given parameters from mailerconfig.
        /// </summary>
        /// <param name="sender">Sender address</param>
        /// <param name="recipients">List of recipient addresses</param>
        /// <param name="subject">subject for the mail</param>
        /// <param name="message">normally formatted message</param>
        /// <param name="monospaceMessage">monospace formatted message, eg. callstack, or code</param>
        /// <param name="applicationName">name of the calling application</param>
        /// <param name="notificationType">type for formatting the mail. possible types are: Failure, Critical, Warning, Healthy, Information</param>
        public static void SendMessageFromTo(
            string sender,
            List<string> recipients,
            string subject,
            string message,
            string monospaceMessage = "",
            string applicationName = "",
            NotificationType notificationType = NotificationType.Information)
        {
            MailerConfig mailerConfig = GetMailerConfig();

            if (string.IsNullOrEmpty(applicationName))
            {
                _applicationName = AssemblyInfoProcessor.GetEntryAssemblyName();
            }

            // Prevent spam if the mail was sent within the amount of minutes before.
            // If value was not set in the mailerconfig file all mails get sent.
            if (CheckIfSpam(subject, mailerConfig.SpamPreventionSubjectAge))
            {
                return;
            }

            string mailTemplate = File.ReadAllText(mailerConfig.MailHtmlTemplatePath);
            MailMessage mailMessage = new MailMessage();
            SmtpClient smtpClient = new SmtpClient();

            message = FixNewLineFormattingForPTag(message);
            mailMessage.Subject = $"{_applicationName} - {notificationType}: {subject}";
            mailMessage.From = new MailAddress(sender);
            foreach (var recipient in recipients)
            {
                mailMessage.To.Add(recipient);
            }
            mailMessage.IsBodyHtml = true;
            mailMessage.Body = mailTemplate
                .Replace("{{NotificationType}}", notificationType.ToString())
                .Replace("{{MachineName}}", Environment.MachineName)
                .Replace("{{AssemblyName}}", AssemblyInfoProcessor.GetEntryAssemblyName())
                .Replace("{{AssemblyVersion}}", AssemblyInfoProcessor.GetEntryAssemblyVersion())
                .Replace("{{ApplicationName}}", _applicationName)
                .Replace("{{Message}}", message)
                .Replace("{{MonospaceMessage}}", monospaceMessage);
            mailMessage.BodyEncoding = Encoding.UTF8;
            mailMessage.SubjectEncoding = Encoding.UTF8;
            smtpClient.Host = mailerConfig.SmtpHost;

            TrySendMessage(smtpClient, mailMessage, message, monospaceMessage);
        }

        /// <summary>
        /// Sends a message as plain text body without Mailtemplate.
        /// Uses mailerconfig parameters.
        /// </summary>
        /// <param name="subject">subject for the mail</param>
        /// <param name="message">normally formatted message</param>
        public static void SendPlainTextMessage(string subject, string message)
        {
            MailerConfig mailerConfig = GetMailerConfig();

            SendPlainTextMessageFromTo(sender: mailerConfig.Sender,
                recipients: mailerConfig.RecipientList,
                subject: subject,
                message: message);
        }

        /// <summary>
        /// Sends a message as mail only aas text body without Mailtemplate.
        /// Uses mailerconfig parameters.
        /// </summary>
        /// <param name="subject">subject for the mail</param>
        /// <param name="message">normally formatted message</param>
        public static void SendPlainTextMessageFromTo(string sender, List<string> recipients, string subject, string message)
        {
            MailerConfig mailerConfig = GetMailerConfig();

            if (CheckIfSpam(subject, mailerConfig.SpamPreventionSubjectAge))
            {
                return;
            }

            MailMessage mailMessage = new MailMessage();
            SmtpClient smtpClient = new SmtpClient();

            mailMessage.Subject = $"{subject}";
            mailMessage.From = new MailAddress(sender);
            foreach (var recipient in recipients)
            {
                mailMessage.To.Add(recipient);
            }
            mailMessage.IsBodyHtml = false;
            mailMessage.Body = $"{message}";
            mailMessage.BodyEncoding = Encoding.UTF8;
            mailMessage.SubjectEncoding = Encoding.UTF8;
            smtpClient.Host = mailerConfig.SmtpHost;
            
            TrySendMessage(smtpClient, mailMessage, message, monospaceMessage);
        }

        /// <summary>
        /// Fixes new line markers in messages like \n, \r or \r\n to br tags for better formatting the text.
        /// </summary>
        /// <param name="message">text with possible new line markers</param>
        /// <returns>text with converted new line markers</returns>
        public static string FixNewLineFormattingForPTag(string message)
        {
            // Check order is important because else it will replace \r\n with two br tags
            if (message.Contains("\r\n"))
            {
                message = message.Replace("\r\n", "<br>");
            }

            if (message.Contains("\n"))
            {
                message = message.Replace("\n", "<br>");
            }

            if (message.Contains("\r"))
            {
                message = message.Replace("\r", "<br>");
            }

            return message;
        }

        /// <summary>
        /// Checks if a mail with given subject was sent within the last minutes as subjectAge.
        /// Reads and writes checked mails in _lastMailSentDict to AppData folder if spamPreventonStored is true.
        /// </summary>
        /// <param name="subject">subject text of the mail</param>
        /// <param name="subjectAge">integer in minutes to prevent spam</param>
        /// <param name="spamPreventionStored">bool if dictionary for recognizing spam shoul be stored</param>
        /// <returns>true, if incoming subject is younger than subjectAge, else false</returns>
        public static bool CheckIfSpam(string subject, int subjectAge = 30, bool spamPreventionStored = false)
        {
            bool isSpam = false;

            // if it is lower or equal to 0, spam prevention is disabled and everything gets sent.
            if (subjectAge <= 0)
            {
                return isSpam;
            }

            if (spamPreventionStored)
            {
                ReadStoredLastMails();
            }

            var subjectHash = subject.GetHashCode();

            if (_lastMailSentDict.TryGetValue(subjectHash, out DateTime lastValue))
            {
                if (lastValue.AddMinutes(subjectAge) < DateTime.Now)
                {
                    lock (_lock)
                    {
                        _lastMailSentDict.Remove(subjectHash);
                        _lastMailSentDict.Add(subjectHash, DateTime.Now);
                    }
                    isSpam = false;
                }
                else
                {
                    lock (_lock)
                    {
                        _lastMailSentDict.Remove(subjectHash);
                        _lastMailSentDict.Add(subjectHash, DateTime.Now);
                    }

                    isSpam = true;
                }
            }
            else
            {
                lock (_lock)
                {
                    _lastMailSentDict.Add(subjectHash, DateTime.Now);
                }

                isSpam = false;
            }

            if (spamPreventionStored)
            {
                WriteStoredLastMails();
            }

            return isSpam;
        }

        /// <summary>
        /// Adds a hash as key with DateTime.Now as value to the _lastMailSentDict if the hash as doesn't exist yet.
        /// </summary>
        /// <param name="subjectHash">hash as key for the dict to add</param>
        public static void AddHashToLastMailSentDict(int subjectHash)
        {
            if (_lastMailSentDict.ContainsKey(subjectHash))
            {
                return;
            }

            _lastMailSentDict.Add(subjectHash, DateTime.Now);
        }

        /// <summary>
        /// Gets the mailerConfig from Config folder.
        /// Sets the CurrentDirectory to AppDomain.CurrentDomain.BaseDirectory for services and tasks to find the Config folder in the execution path.
        /// </summary>
        /// <returns></returns>
        private static MailerConfig GetMailerConfig()
        {
            Directory.SetCurrentDirectory(AppDomain.CurrentDomain.BaseDirectory);
            string mailerConfigJson = File.ReadAllText(@"Config\MailerConfig.json");
            MailerConfig mailerConfig = JsonSerializer.Deserialize<MailerConfig>(mailerConfigJson);
            return mailerConfig;
        }

        /// <summary>
        /// Tries to send a perared mailMessage with the perared smtpClient.
        /// If sending fails, it throws an exception filled with the given message and monospaceMessage texts or "not provided" if null.
        /// This should enable preserving the alertMail messages in case of failuere.
        /// </summary>
        /// <param name="smtpClient">prepared smtpClient from calling method</param>
        /// <param name="mailMessage">prepared mailMessage from calling method</param>
        /// <param name="message">message part of the given mailMessage to be preserved as exception text</param>
        /// <param name="monospaceMessage">monospaceMessage part of the given mailMessage to be preserved as exception text</param>
        private static void TrySendMessage(SmtpClient smtpClient, MailMessage mailMessage, string message = null, string monospaceMessage = null)
        {
            try
            {
                smtpClient.Send(mailMessage);
            }
            catch (Exception ex)
            {
                message = message ?? "not provided";
                monospaceMessage = monospaceMessage ?? "not provided";

                string errorMessage = new StringBuilder()
                    .AppendLine("Error while sending an AlertMail!")
                    .AppendLine()
                    .AppendLine($"Mailer Exception: {ex.Message}")
                    .AppendLine($"Mail Message Text: {message}")
                    .AppendLine($"Mail MonospaceMessage Text:")
                    .AppendLine($"{monospaceMessage}")
                    .ToString();

                EventLogWriter.WriteApplicationEventEntry(errorMessage, System.Diagnostics.EventLogEntryType.Error);

                throw new Exception(errorMessage);
            }
        }

        /// <summary>
        /// Reads the stored values for last mails from path.
        /// Doesn't overwrite the existing empty Dictionary in _lastMailSentDict field if the stored file in path doesn't exist.
        /// </summary>
        private static void ReadStoredLastMails()
        {
            var path = Path.Combine(AppDataCommonPathProcessor.GetCompanyFolderPath(_applicationName), "lastMailSent.json");
            if (File.Exists(path))
            {
                var lastMailSentJson = File.ReadAllText(path);
                lock (_lock)
                {
                    _lastMailSentDict = JsonSerializer.Deserialize<Dictionary<int, DateTime>>(lastMailSentJson);
                }
            }
        }

        /// <summary>
        /// Writes the in memory Dictionary _lastMailSentDict as json to the given path.
        /// </summary>
        /// <param name="path">path to json file with stored last mails</param>
        private static void WriteStoredLastMails()
        {
            var path = Path.Combine(AppDataCommonPathProcessor.GetCompanyFolderPath(_applicationName), "lastMailSent.json");
            File.WriteAllText(path, JsonSerializer.Serialize(_lastMailSentDict));
        }
    }
}
```

## Utility class to write Windows EventLog entries

```csharp
using System.Diagnostics;

namespace Production.Utilities
{
    public static class EventLogWriter
    {
        private const string EventLogApplicationSubSource = ".NET Runtime";
        private const int EventLogDefaultId = 1026;

        /// <summary>
        /// Writes a given message to the Application Windows EventLog.
        /// The optional parameters can be overwritten with custom values.
        /// </summary>
        /// <param name="message">message to be written as log entry</param>
        /// <param name="eventLogEntryType">overwrites the default type of Information to a custom set type</param>
        /// <param name="sourceName">Overwrites the default ".NET Runtime". Custom values have to exist, or this throws an exception.</param>
        /// <param name="eventId">OVerwrites the default id of 1026 to a custom id. keeping the default is recommended, else chose an id above 1000</param>
        public static void WriteApplicationEventEntry(
            string message,
            EventLogEntryType eventLogEntryType = EventLogEntryType.Information,
            string sourceName = EventLogApplicationSubSource,
            int eventId = EventLogDefaultId)
        {
            using (EventLog eventLog = new EventLog())
            {
                eventLog.Source = sourceName;
                eventLog.WriteEntry(message, eventLogEntryType, eventId);
            }
        }
    }
}
```

*More about this in [Article about writing eventLog](/docs/csharp/write-win-eventlog-entry.md).*


## Utility class to access the AppData folder with CompanyName and AssemblyName

```csharp
using System;
using System.IO;
using System.Reflection;

namespace Production.Utilities
{
    public static class AppDataCommonPathProcessor
    {
        /// <summary>
        /// Gets a common path for settings and files for the application in the LocalAppData folder, combined with company name and application name.
        /// Creates the directories of the path unless they don't exist.
        /// </summary>
        /// <param name="applicationName">A subfolder with the given application string gets created if provided.</param>
        /// <returns>Path to appsettings folder</returns>
        public static string GetCompanyFolderPath(string applicationName = "")
        {
            string company = ((AssemblyCompanyAttribute)Attribute.GetCustomAttribute(Assembly.GetEntryAssembly(), typeof(AssemblyCompanyAttribute), false)).Company;
            company = string.IsNullOrEmpty(company) ? "Your_Company_Name" : company.Replace(' ', '_');

            string companyPath = Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData), company, applicationName);
            Directory.CreateDirectory(companyPath);
            return companyPath;
        }

        /// <summary>
        /// Clears files from a directory where the file name matches a given string phrase.
        /// </summary>
        /// <param name="folderPath">path to be cleared</param>
        /// <param name="match">string to recognize files to be deleted, e.g. ".pdf"</param>
        /// <param name="ageInDays">number of days to hold old files</param>
        public static void ClearFilesFromPath(string folderPath, string match, int ageInDays = 5)
        {
            string[] files = Directory.GetFiles(folderPath);

            foreach (var file in files)
            {
                FileInfo fi = new FileInfo(file);
                if (fi.LastWriteTime < DateTime.Now.AddDays(-ageInDays) && fi.Name.Contains(match))
                {
                    fi.Delete();
                }
            }
        }
    }
}
```


## Class to read assembly infos

```csharp
using System;
using System.IO;
using System.Reflection;

namespace Production.Utilities
{
    public static class AssemblyInfoProcessor
    {
        /// <summary>
        /// Gets the entry assembly name.
        /// </summary>
        /// <returns>Name of the entry assembly</returns>
        public static string GetEntryAssemblyName()
        {
            return Assembly.GetEntryAssembly().GetName().Name;
        }

        /// <summary>
        /// Gets the entry assembly version.
        /// </summary>
        /// <returns>Verson of the entry assembly</returns>
        public static string GetEntryAssemblyVersion()
        {
            return $"{Assembly.GetEntryAssembly().GetName().Version}";
        }
    }
}
```


## HTML and CSS Mailtemplate

```html
<!DOCTYPE html>
<html lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title>Email Alert</title>
    <style>
        body {
            background-color: #f0f0f0;
            font-family: Calibri, sans-serif;
            color: #404040;
        }

        .center {
            text-align: center;
        }

        td {
            padding: 20px 50px 30px 50px;
        }

        small,
        .small {
            font-size: 12px;
        }

        a,
        a:hover,
        a:visited {
            font-size: 12px;
            color: #000000;
            text-decoration: underline;
        }

        h1,
        h2 {
            font-size: 22px;
            color: #404040;
            font-weight: normal;
        }

        p {
            color: #606060;
            white-space: pre;
        }

        pre {
            font-family: monospace;
            white-space: pre;
        }

        .general {
            background-color: white;
        }

        .Failure {
            border-top: 20px #b02020 solid;
            background-color: #db9c9b;
        }

        .Critical {
            border-top: 20px #c05050 solid;
            background-color: #e2afae;
        }

        .Warning {
            border-top: 20px #c08040 solid;
            background-color: #e0c4aa;
        }

        .Healthy {
            border-top: 20px #80c080 solid;
            background-color: #c6e2c3;
        }

        .Information {
            border-top: 20px #50a0c0 solid;
            background-color: #b5d5e2;
        }

        .Failure p {
            color: #3d120f;
            padding: 0;
        }

        .Critical p {
            color: #3d211f;
        }

        .Warning p {
            color: #44311c;
        }

        .Healthy p {
            color: #364731;
        }

        .Information p {
            color: #273c47;
        }
    </style>
</head>
<body style="margin: 0; padding: 0">
    <table style="border: none" cellpadding="0" cellspacing="0" width="100%">
        <tr>
            <td style="padding: 15px 0">
                <table style="border: none;" cellpadding="0" cellspacing="0" width="99%" class="content">
                    <tr>
                        <td class="general">
                            <h1>Email Alert: {{ApplicationName}}</h1>
                        </td>
                    </tr>
                    <tr>
                        <td class="{{NotificationType}}">
                            <h1>Notification Level: {{NotificationType}}</h1>
                            <p>MachineName: <b>{{MachineName}}</b> | AssemblyName: <b>{{AssemblyName}}</b> | AssemblyVersion: <b>{{AssemblyVersion}}</b><br />{{Message}}</p>
                            <pre>{{MonospaceMessage}}</pre>
                        </td>
                    </tr>
                    <tr>
                        <td class="general">
                            <p><a href="https://www.yourdomain.de/">Your Website</a> | <a href="https://dev.azure.com/">DevOps</a></p>
                        </td>
                    </tr>
                </table>
            </td>
        </tr>
    </table>
</body>
</html>
```


## JSON config

```json
{
  "MailHtmlTemplatePath": "MailTemplates\\AlertMail.html",
  "SmtpHost": "mailserver.yourdomain",
  "Sender": "AlertMail@yourdomain",
  "SpamPreventonSubjectAge": 30,
  "RecipientList": [
    "somebody@yourdomain"
  ]
}
```
