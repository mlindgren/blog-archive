---
layout: post
title: "Fun with the Microsoft Band SDK Preview"
date: 2015-03-14 02:07:34 -0700
comments: true
categories: [Coding]
---

First of all, happy [Pi Day](http://en.wikipedia.org/wiki/Pi_Day)! Using the standard United States
date format, today is 3/14/15, which makes it the most significant Pi Day this century.

Anyway, I've spent the past few hours tinkering with the
[Microsoft Band SDK Preview](http://developer.microsoftband.com/), and although I think it's
unfortunate that there's no SDK for writing apps which actually run on the Band itself, you can
still accomplish some pretty cool things with the phone SDK. Unfortunately, this being a preview
release, the documentation is a bit sparse, and I had some trouble figuring out how to get things
working. Thus, I wanted to share some of what I've learned. If you prefer to dive right into the
code, I've got a small C# sample application [on GitHub](https://github.com/mlindgren/BandDemo).

Standard disclaimer: although I am a Microsoft employee, everything I am posting here and on GitHub
is my own personal work done on my own time. None of it should be considered as officially
representative of Microsoft in any respect. While I will do my best to answer any questions I can
related to the Band SDK Preview, I cannot provide official support for this or any other Microsoft
product.

Let's get started. If you're following
[the documentation](http://developer.microsoftband.com/docs/MicrosoftBandSDKPreview.pdf), the first
few steps describe how to connect to the Band, as one would expect. The sample code provided creates
a connection within a `using` block. This ensures that the connection is disposed of as soon as
control leaves the block, which is sensible since maintaing a connection to the Band, especially
when subscribed to multiple sensors, can severely impact its battery life. However, this approach
can also be a bit misleading, because if you _do_ want to collect data from the Band over some
interval of time, you will only be able to do so during the lifetime of the `IBandClient` returned
by `ConnectAsync`. As soon as the `IBandClient` is disposed of, your connection will be closed and
you will no longer receive sensor data.

Since you'll typically only be connected to one Band at a time, the easiest way to solve this
problem is to store the `IBandClient` as a static member variable of your `Application` class.
(Note: I don't guarantee that this is the _optimal_ solution. I am by no means an expert here.) This
also makes it easy to access your Band client from different pages within your application, and to
unsubscribe from sensor data when you no longer need it so that you won't drain the Band's
battery.<!-- more -->

[**Updated March 17, 2015:** In my original post, I was missing a call to `Dispose()` before setting
the `IBandClient` member to `null`. As Phil pointed out in the comments, this is a bad practice,
because garbage collection of the `IBandClient` may not happen immediately, in which case the
connection to the Band would be left open for some time. During that time, other applications would
be prevented from connecting to the Band. I have corrected this error in my code below, and in my
project on GitHub. If you are only connecting to the Band for a short time, you can use a `using`
block instead, as discussed above, which will ensure that `Dispose()` is called for you. Thanks to
Phil for the correction.]

{% codeblock lang:csharp App.xaml.cs %}
using Microsoft.Band;

namespace BandDemo
{
  public sealed partial class App : Application
  {
    public static IBandClient BandClient
    {
      get; set;
    }

    // ...

    private async void OnSuspending(object sender, SuspendingEventArgs e)
    {
        var deferral = e.SuspendingOperation.GetDeferral();

        if(App.BandClient != null)
        {
            // Unsubscribe from sensor data so that we don't drain the Band's battery when
            // the app is not in use.
            await App.BandClient.SensorManager.Accelerometer.StopReadingsAsync();
            await App.BandClient.SensorManager.Gyroscope.StopReadingsAsync();
            await App.BandClient.SensorManager.HeartRate.StopReadingsAsync();

            // Call Dispose to close the connect to the Band
            App.BandClient.Dispose();

            // We are done with this client, so assign null to the member variable so the
            // IBandClient can be garbage collected
            App.BandClient = null;
        }

        deferral.Complete();
    }
  }
}
{% endcodeblock %}

{% codeblock lang:csharp MainPage.xaml.cs %}
using Microsoft.Band;
using Microsoft.Band.Sensors;

namespace BandDemo
{
  public sealed partial class MainPage : Page
  {

    // ...

    protected async override void OnNavigatedTo(NavigationEventArgs e)
    {
      IBandInfo[] pairedBands = await BandClientManager.Instance.GetBandsAsync();

      try
      {
          App.BandClient = await BandClientManager.Instance.ConnectAsync(pairedBands[0]);

          // Do work after successful connect
          System.Diagnostics.Debug.WriteLine("Connected!");
      }
      catch(Exception)
      {
        System.Diagnostics.Debug.WriteLine("Connection failed!");
      }
    }
  }
}
{% endcodeblock %}

The next thing you'll notice in the documentation is that it recommends that you query the device
for available reporting intervals for each sensor. It's fairly easy to miss that setting the
reporting interval is _completely optional_; you can feel free to skip this step unless you really
need fine-grained control over how frequently you get updates.

Finally, the documentation describes how to subscribe to sensor data by adding your callback to a
sensor's `ReadingChanged` property. When your callback is executed, you might want to update your
app's UI to indicate the new reading (indeed, the documentation itself suggests this). However, this
is another gotcha because in Windows (Phone) Store apps, "all UI components share the same [UI]
thread," ([reference](https://msdn.microsoft.com/en-us/library/windows/apps/hh994635.aspx)), and
methods on this thread cannot be directly invoked by background threads. If you try to do so, an
exception will be raised. This will of course be no surprise to those of you already familiar with
Store apps (or even with .NET circa 2006), but for the rest of us, it can be a bit confusing.
Luckily, the solution is simple: you just need to use `Dispatcher.RunAsync` to do your UI updates.
So, putting it all together, here's how you subscribe to sensor data and update the UI when it
changes:

{% codeblock lang:csharp %}
App.BandClient.SensorManager.Accelerometer.ReadingChanged += async (sender, args) =>
{
  await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () =>
  {
    AccelerometerXValue.Text = args.SensorReading.AccelerationX.ToString("###00.00");
    AccelerometerYValue.Text = args.SensorReading.AccelerationY.ToString("###00.00");
    AccelerometerZValue.Text = args.SensorReading.AccelerationZ.ToString("###00.00");

    System.Diagnostics.Debug.WriteLine("Got accelerometer event!");
  });
};

await App.BandClient.SensorManager.Accelerometer.StartReadingsAsync();
{% endcodeblock %}

That's it for now! If you have any questions, leave a comment and I'll do my best to answer them.
