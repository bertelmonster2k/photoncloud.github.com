---
layout: article
title: Dashboard Customization
categories: [photon-server, getting_started]
tags: [how-to, setup, installation]
---
{% include globals %}

## Photon Dashboard: Customizing

In the previous part of the [Photon Dashboard
tutorial](/dashboard), we have discussed how to install and
display Photon's built in Performance Counters. Now we are going to see
how we can easily create our own, custom counters and adjust the layout
of the Photon Dashboard to our needs.

Let's assume you have a custom application that is extending the [Lite
application](/WhatsInPhoton3), and want to add another counter -
for example, imagine that you have a custom operation "SendMessage" and
that you want to know how many messages are sent per second.

## Implementing custom performance counters

At first, your application needs to track the desired values. We are
going to use the in-memory counters from ExitGames.Diagnostics.Counter
for this purpose.

Make sure that your application has a reference to the ExitGamesLibs.dll
from the /lib folder:

<figure>
<img src="{{ IMG }}/DashboardCustomizing-Reference.png" />
<figcaption>Referencing the ExitGamesLibs.dll</figcaption>
</figure>

### Counter declaration

The next step is to create a counter definition class. Add
"using"-directives for ExitGames.Diagnostics.Counter and
ExitGames.Diagnostics.Monitoring and declare your custom counter, like
shown below. Add a "PublishCounter" attribute with a unique name; this
makes sure that the counter value will be published by the the Counter
Publisher (see below).

~~~~ {.code}

using ExitGames.Diagnostics.Counter;
using ExitGames.Diagnostics.Monitoring;

public static class MyCustomCounter
{
  [PublishCounter(MessagesPerSecond")]
  public static readonly CountsPerSecondCounter MessagesPerSecond = new CountsPerSecondCounter();
}
~~~~

See the appendix for a list of available counter types.

### Counter Usage

In the appropriate place in your code, increment or decrement the
counter. For example, to count the number of sent messages per second,
add the following code to the method that handles your "SendMessage"
operation:

~~~~ {.code}

protected virtual void HandleSendMessageOperation(LitePeer peer, OperationRequest operationRequest)
{
  // [... operation logic ...]

  // increment performance counter:
  MyCustomCounter.MessagesPerSecond.Increment();
}
~~~~

That's all - now you have done everything that is required to collect
your performance data.

### Counter Publisher initialisation

The next step is to publish the collected performance data to the Photon
Dashboard.

For this, you need to initialize a CounterPublisher during the start-up
of our application. The Setup() method of your application is a good
place for this.

This example shows how to initialize the Counter Publisher and add your
custom counter definition class. You can choose an arbitrary name for
the second parameter - it is used as the category in which the counter
will be displayed by the Dashboard later.

~~~~ {.code}

protected override void Setup()
{
  // [... setup logic  ...]

  // initialize counter publisher and add your custom counter class(es):
  if (PhotonSettings.Default.CounterPublisher.Enabled)
  {
    // counters for the photon dashboard

    CounterPublisher.DefaultInstance.AddStaticCounterClass(typeof(MyCustomCounter), "MyApplication");
    CounterPublisher.DefaultInstance.Start();
  }
}
~~~~

### Counter Publisher configuration

The last step is to add the appropriate configuration for the counter
publisher. Configure it to send data to the IP / Port where your Photon
Dashboard is running. Remember that multicasting is only working in the
same subnet; to publish performance data between different networks, use
the concrete IP and make sure that the port is opened up in the
firewall.

~~~~ {.code}

  <configSections>

    <section name="Photon" type="Photon.SocketServer.Diagnostics.Configuration.PhotonSettings, Photon.SocketServer"/>
  </configSections>

  <Photon>
    <CounterPublisher
      enabled="True"

      endpoint="255.255.255.255:40001"
      protocol="udp"
      sendInterface=""
      updateInterval="1"
      publishInterval="10"/>

  </Photon>
  
~~~~

## Understanding the Photon Dashboard

In the previous steps we have made sure that our application tracks and
publishes the required performance data. Now we need to make sure that
the Photon Dashboard, which receives the published\
 data, can display the counter values.

To extend the Photon Dashboard with your custom counters, it's helpful
to understand roughly how the Dashboard is working internally. The
following schema should give you a good idea.

<figure>
<img src="{{ IMG }}/DashboardCustomizing-Schema.png" />
<figcaption>Components of the Photon Dashboard</figcaption>
</figure>

A counter receiver is listening on the specified UDP port for the
counter data published by Photon. It stores the received values in RRD
("round robin database") files. These are files of a fixed size (ie.,
after they are initially created, they are not growing - no matter how
long the Dashboard is running or how much data it receives) and store
values for a fixed period of a time - after that, the oldest entries are
overwritten.

At certain intervals, the Dashboard renders image files from the RRD
files. A configuration file called Graphs.xml specifies how these images
should look like. We'll discuss the configuration options further in the
next section.

The Photon Dashboard contains an internal HTTP server, so that you can
easily access and browse the performance counter graphs on a web site,
which can also be customized to your needs.

## Extending the Photon Dashboard

### Counter Receiver

See the first part of the [Photon Dashboard
tutorial](/dashboard)to learn how to configure the Counter
Receiver, e.g. the ports / IPs it is listening to.

### Graph renderer

For each graph that we want to create from our custom counters, we need
to add an item to the Graphs.xml config file. You can find that file in
/bin\_tools/dashboard.

The items in the Graphs.xml file look like this:

~~~~ {.code}

<add sender="*" fileName="MyApplication\MessagesPerSecond" title="Messages sent / sec" timeSpan="00:10:00">

 <Items>
  <add fileName="MyApplication.MessagesPerSecond.rrd" dataSource="GAUGE" cf="AVERAGE" type="Area" color="Blue"/>

 </Items>
</add>
~~~~

In most cases, you only need to adjust the image filename, the
datasource (rrd) file name, and the title.

However, here is an explanation of all available attributes, in case you
want to create some more advanced graphs:

-   **sender** - To display counters from all source servers in a single
    graph, set to *sender="\*"*. If you want to display only counters
    from specific servers, add the hostname, e.g.: *sender="ServerA"*
-   **filename** - The image file for your counter application will be
    created inside the "Images" folder. You can specify a subfolder like
    "MyApplication\\MessagesPerSecond.jpg", if you want your image to be
    shown in an own category. Folder- and filenames are arbitrary.
-   **title** - an arbitrary title that is shown in your graph.
-   **timeSpan** - The grapsh shows counter data for the specified
    timespan, e.g. for the last 10 minutes. The shorter the time span
    is, the more fine-grained the displayed values are.
-   **lowerLimit** - Only values that exceed the specified value are
    displayed. The y-axis will start at the lowerLimit.
-   **upperLimit** - Only values that are below the specified value are
    displayed. The y-axis will stop at the upperLimit. E.g., to show a
    percentage scale, you can limit the x-Axis to 0-100 by setting
    *lowerLimit="0"* and *upperLimit="100"*

Each graph can show values from one or many data source items. These are
the available attributes for the elements in the *"Items"* section:

-   **filename** - The name of the RRD data source file, where the
    received counter values are stored. The data sources reside in the
    "PerfData" folder and are named as "AppName.CounterName.rrd", where
    the AppName is the name that was specified during the counter
    publisher initialization (here: "MyApplication") and the CounterName
    is the name that was specified in the "PublishCounter" attribute
    (here: "MessagesPerSecond").
-   **dataSource** - Choose the format of the datasource. Performance
    Counter values are stored in two different formats: "GAUGE"
    (default) and "COUNTER". The "GAUGE" type data source stores the
    actual performance counter value. the "COUNTER" type data source
    stores the rate of change in a specific time frame, assuming the
    value never decreases. (A traffic counter on a network interface
    card would be a good example). Typically, you should use
    *dataSource="GAUGE"* here.
-   **cf** - The consolidation function. Supported values are "AVERAGE",
    "MINIMUM", "MAXIMUM", "LAST".
-   **type** - The graph type. Supported values are "Area", "Line1",
    "Line2", "Line3" and "Stack".
-   **color** - Display the data from this data source in the defined
    colour.

For a better understanding how graphs are created from RRD data sources,
see
[http://www.mrtg.org/rrdtool/tut/rrd-beginners.en.html](http://www.mrtg.org/rrdtool/tut/rrd-beginners.en.html).

### HTTP Server

Information on the configuration options for the HTTP server - e.g., on
which port and IP it should run - can be found in the first part of this
tutorial: (tdb: link)

If you want to customize the look of the Dashboard web UI, you can make
changes to the html file or the .css stylesheets in
/deploy/bin\_tools/dashboard/web.

## Appendix: List of Counters from ExitGames.Diagnostics.Counter

This section discusses the available types of in-memory counters from
the ExitGames.Diagnostics.Counter namespace in greater detail.

All counters implement the ICounter interface:

~~~~ {.code}

namespace ExitGames.Diagnostics.Counter
{
    public interface ICounter
    {
        string Name { get; }

        // Gets the type of the counter.

        CounterType CounterType { get; }

        // Gets the next value. 
        float GetNextValue();

        // Increments the counter by one and returns the new value.
        long Increment();

        // Increments the counter by a given value and returns the new value.
        long IncrementBy(long value);

        // Decrements the counter by one and returns the new value.
        long Decrement();
    }
}
~~~~

As you see, there are mainly methods to increment / decrement the
counter value.

GetNextValue() is the method to return the actual "value" of the
counter. This can be a calculated value - for example, when you call
GetNextValue() on an AverageCounter, the counter calculates the average
of all Increments / Decrements since the previous call to GetNextValue()
and returns that value. **GetNextValue() resets internal values - you
should never call it in your code manually, otherwise the published data
will be incorrect.** The CounterPublisher makes calls to GetNextValue()
at regular intervals.

#### NumericCounter

This is a very straight-forward counter. It holds a single value;
GetNextValue() returns that single value and does **not** reset the
counter.

**Example**: At the start of a 10-second-interval, the NumericCounter
has a value of 5. It is incremented 2 times, with values of 1 and 6, and
decremented once with a value of 3, resulting in a value of 5+1+6-3 =
**9** for this interval.

**Typical use-case**: The number of users that are currently logged on
to your system.

**Simple code example**:

~~~~ {.code}

public void AddToCache(ArrayList items) 
{
   foreach (var item in items) 
   {
     Cache.Insert(item);
   }

   MyCacheItemCounter.IncrementBy(items.Count);
}

public void RemoveFromCache(ArrayList items) 
{
   foreach (var item in items) 
   {
     Cache.Remove(item);
   }

   MyCacheItemCounter.IncrementBy( -items.Count);
}

// When the CounterPublisher calls MyCacheItemCounter.GetNextValue(), it always returns the current number of items in cache. 
~~~~

#### AverageCounter

The AverageCounter tracks how often it was called since the last reset,
and can be incremented by any value. It returns the average amount per
call in the current interval (= total amount / \# of calls).

**Example**: In a 10-second-interval, the AverageCounter is incremented
4 times, with values of 1,2,4,5. This is 12 in total, resulting in an
average of 12/4 = **3** for this interval.

Typical use-case: The average exeuction time of a certain operation, or
the average text length per sent message.

**Simple code example:**

~~~~ {.code}

public void WinGame(Player player, int coins) 
{
   player.Account.Add(coins);

   AverageCoinsWonCounter.IncrementBy(coins);
}

public void LoseGame(Player player, int coins) 
{
   player.Account.Remove(coins);

   AverageCoinsWonCounter.IncrementBy( -coins);
}

// When the CounterPublisher calls AverageCoinsWonCounter.GetNextValue(), it returns the average amount a player has won in the interval since the previous call to GetNextValue(). 
// This amount might as well be negative. The internal counter value is reset afterwards - i.e., if no Game was won or lost between two calls to GetNextValue(), the average is 0.
~~~~

#### CountsPerSecondCounter

The CountsPerSecondCounter holds a single value, which can be
incremented / decremented by any value. It returns the average amount
per second in the last interval. You only need to increment / decrement
the counter, it converts these calls in "per second" values
automatically.

**Example**: In a 10-second-interval, the CountsPerSecondCounter is
incremented 3 times, with values of 10, 20, and 70. This is 100 in
total, resulting in a value of **10 per second** for this interval.

**Typical use-case**: The number of sent bytes per second on a network
interface, or the number of times a specific action is taken per second.

**Simple code example:**

~~~~ {.code}

public void RaiseEvent() 
{
   // do something to raise an event

   EventsPerSecondCounter.Increment(); 
}

// RaiseEvent can be called at any time - multiple times per second, or only once every few seconds, and requires no fixed interval.
// When the CounterPublisher calls EventsPerSecondCounter.GetNextValue(), the counter checks how many seconds have passed since the the last call to GetNextValue(). 
// It calculates it's return value as  "CurrentValue  / SecondsPassed".  The internal value is reset when GetNextValue() is called.  
~~~~

#### PerformanceCounterReader

The purpose of the PerformanceCounterReader is to read values from
underlying Windows Performance Counters. It is a read-only counter and
can not be incremented / decremented. This is failsaife, i.e. no errors
are thrown if the performance counter does not exist or access is not
allowed.

#### WindowsPerformanceCounter

The WindowsPerformanceCounter is a wrapper class for Windows Performance
Counters; it provides read / write access to the underlying counter. The
performance counter need to exist and the reading / writing application
needs to have the appropriate rights, otherwise an exception is raised.

For more information on Windows Performance Counters, the MSDN is a good
starting point:
[http://msdn.microsoft.com/en-us/library/system.diagnostics.performancecounter.aspx](http://msdn.microsoft.com/en-us/library/system.diagnostics.performancecounter.aspx)
