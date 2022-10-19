---
layout: page
title: C# - Delegate and EventHandler
parent: C#
---

# Delegate and EventHandler

Delegates and EventHandler let you decouple your classes and write multiple consuming subscriber methods to be invoked on a raised event. Imagine you have a video encoder class and you want to react on the finished video encoding process with sending some information about the finished process. With EventHandler you can implement this without having to change the video encoder class for each new messaging service you want to add in the future, but just register a new subscriber to the publisher to get informed about the event.


## Main ties it all together

```csharp
private static void Main(string[] args)
{
    var video = new Video() { Title = "Some Video" };
    var videoEncoder = new VideoEncoder();      // Publisher publishes events
    var mailService = new MailService();        // Subscriber with called method
    var messageService = new MessageService();  // Subscriber with called method

    videoEncoder.VideoEncoded += mailService.OnVideoEncoded;    // register the subscriber
    videoEncoder.VideoEncoded += messageService.OnVideoEncoded; // register another subscriber

    videoEncoder.Encode(video);

    Console.WriteLine("Press any button to exit ...");
    Console.ReadKey();
}
```

The subscribing mailService and even another subscriber, the messageService register their implementation of the `OnVideoEncoded` method to the `videoEncoder.VideoEncoded` EventHandler.


## VideoEncoder class as publisher

```csharp
internal class VideoEncoder
{
    // Fully defined delegate and event for handling the objects
    //public delegate void VideoEncodedEventHandler(object source, VideoEventArgs e);
    //public event VideoEncodedEventHandler VideoEncoded;

    // This is the same but shorter:
    public event EventHandler<VideoEventArgs> VideoEncoded;

    public void Encode(Video video)
    {
        Console.WriteLine("Encoding video ...");
        Thread.Sleep(1000);

        // Call the event method with the data
        OnVideoEncoded(video);
    }

    /// <summary>
    /// Eventpublisher invokes the subscriber methods.
    /// Follow the conventions of having a protected virtual void method to be called.
    /// </summary>
    /// <param name="video">data to be transmitted in the event, the video object in this case</param>
    protected virtual void OnVideoEncoded(Video video)
    {
        // Checks if there are any subscribers listening to the event
        //if (VideoEncoded != null)
        //    VideoEncoded(this, EventArgs.Empty);

        // This does the same but in only one line
        VideoEncoded?.Invoke(this, new VideoEventArgs() { Video = video });
    }
}
```

The VideoEncoder class holds the EventHandler definition, the event publishing method `OnVideoEncoded` and the processing `Encode` method which calls the event publishing method on its end. `OnVideoEncoded` invokes the event to be handled and published to all subscribers. It even has some data (the video object) to be transmitted. `.Invoke(<source>, <EventArgs>)` comes with the source/the caller and the EventArgs with optional data. We have a specialized class for the Eventargs, the VideoEventArgs, which derives from Eventargs and can be transmitted as such.


## MailService as a Subscriber 

```csharp
internal class MailService
{
    public void OnVideoEncoded(object source, VideoEventArgs e)
    {
        Console.WriteLine($"MailService: Video was encoded with title '{e.Video.Title}'");
    }
}
```

The MailService is registered as one of the subscribers in Main and gets its Method `OnVideoEncoded(object source, VideoEventArgs e)` invoked by the EventHandler on raise. This method now gets the EventArgs, or in this case the specialized VideoEventArgs and can process this data.

The MessageService is another subscriber, who could act in a different way, e.g. send a text message. Even if you would add further subscribers with a `OnVideoEncoded(object source, VideoEventArgs e)` method, you wouldn't have to change the VideoEncoder, but could extend the functionality of the application. This decouples the classes.