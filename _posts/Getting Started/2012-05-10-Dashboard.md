---
layout: article
title: Photon Control 
categories: [photon-server, getting_started, references]
tags: [setup, installation]
---
{% include globals %}

## Photon Dashboard

Photon makes use of Windows Performance Counters as well as custom
in-memory counters to track server performance and statistics, e.g. CPU
load, memory usage, current connection count, operation execution time
etc.

To display these counters, either Windows Performance Monitor or the
Photon Dashboard can be used. The Dashboard is a simple application that
ships with Photon, integrates a small web server and displays all
counters on a monitoring web site. Photon is publishing it's counter
values to the Dashboard via UDP, so that the Photon Dashboard can run on
a different server and display counters from various Photon instances.

Photon performance counters are easily extensible and the display
options can be fully customized.

<figure>
<img src="{{ IMG }}/Dashboard-Overview.png" />
<figcaption>Overview: The Photon Dashboard</figcaption>
</figure>

Photon's performance counters
-----------------------------

### In-Memory counters

Photon's applications are using in-memory performance counters to track
application-specific statistics, e.g. the number of current games.
These counters don't show up in the Windows Performance monitor, but
they are broadcasted to the Photon Dashboard.

Check the 2nd part of this tutorial for more information on Exit Games'
performance counters and how to add your own performance counters to
your application: [Photon Dashboard
Customizing](/dashboardcustomizing)

### Windows Performance counters

As the Photon core is written in C++, it can not make use of Exit Games'
in-memory counters. To track performance data, it relies on native
Windows Performance counters instead.

If you want to make use of these counters, you need to install them
before you start Photon. This can be done in Photon Control:

<figure>
<img src="{{ IMG }}/Dashboard-InstallCounter.png" />
<figcaption>Windows Performance Counter Installation</figcaption>
</figure>

Check Windows' Performance Monitor to see the installed counters:

<figure>
<img src="{{ IMG }}/Dashboard-PerformanceMonitor.png" />
<figcaption>Photon's counter show up at the Performance Monitor</figcaption>
</figure>

Photon is writing data to these counters automatically once it is
started.

Publish counters to the Dashboard
---------------------------------

### Publishing In-Memory Counters

In order to display the counters at the Photon Dashboard, the counter
data needs to be published on a specific UDP port, on which the
Dashboard is listening for the data.

The [Lite application](/WhatsInPhoton3) is making use of a
"CounterPublisher" class to broadcast the in-memory counter data. It can
be enabled / disabled, and it's port and IP can be configured in the
/deploy/Lite/bin/Lite.dll.config:

~~~~ {.code}
  
  <configSections>
    <section name="Photon" type="Photon.SocketServer.Diagnostics.Configuration.PhotonSettings, Photon.SocketServer"/>

  </configSections>

  <Photon>
    <CounterPublisher enabled="True" endpoint="255.255.255.255:40001" protocol="udp" sendInterface="" updateInterval="1" publishInterval="10" />

  </Photon>
  
~~~~

This example is using a multicast IP - please note that you need to
define specific IPs if Photon and Photon Dashboard are in different
networks.

### Publishing Windows Performance Counters

The native Photon core can not publish it's counters directly, because
it is using Windows Performance counters and not Exit Games' in-memory
counter. Therefore, Photon ships with a separate "Counter Publisher"
application, which acts as a kind of proxy: it reads the values from
Photon's Windows Performance counters, temporarily adds them to
in-memory counters and publishes them.

In addition to the Photon core counters, the CounterPublisher
application reads and broadcasts some Windows system counters, like the
CPU load, Memory usage etc.

The CounterPublisher application is enabled by default; it's publishing
behavior can be configured as described above, the settings can be found
in the /deploy/CounterPublisher/bin/CounterPublisher.dll.config.

The following schema shows which types of performance counters are used
by the different parts of Photon, and how they are published:

<figure>
<img src="{{ IMG }}/Dashboard-PhotonSchema.png" />
<figcaption>Photon's Performance Counters</figcaption>
</figure>

The Photon Dashboard
--------------------

The Photon Dashboard is a service that receives the performance counter
data which is published by Photon's CounterPublisher. It's making use of
the RRDTool library (see
[http://oss.oetiker.ch/rrdtool/](http://oss.oetiker.ch/rrdtool/) for
more info) to store and display the data and it comes with an internal
webserver to render HTML sites with performance graphs.

### Installation

The Dashboard can be installed and started from Photon control:

<figure>
<img src="{{ IMG }}/Dashboard-InstallDashboard.png" />
<img src="{{ IMG }}/Dashboard-StartDashboard.png" />
<figcaption>Install and Start the Dashboard from Photon Control</figcaption>
</figure>

### Web UI

The Dashboard comes with a built-in web server. By default, you can
acess it on http://localhost:8088/, or you can use the Photon control to
open the Dashboard:

<figure>
<img src="{{ IMG }}/Dashboard-OpenUI.png" />
<figcaption>Access the Dashboard UI from Photon Control</figcaption>
</figure>

In the upper left corner of the Dashboard site, you'll find a list of
all machines that are publishing counter data to this Dashboard, as well
as a list of all applications from which the published counters are
originating. Choose the appropriate machine / application to find the
counters you are interested in.

<figure>
<img src="{{ IMG }}/Dashboard-UI.png" />
<figcaption>The Photon Dashboard UI</figcaption>
</figure>

### Configuration options

Use the configuration file at
/deploy/bin\_tools/dashboard/PhotonDashboard.exe.config to configure
various Dashboard settings.

The IP and port for access to the Photon Dashboard web interface can be
set in the "appSettings" section:

~~~~ {.code}

<appSettings>
    <add key="ImagePath" value="Images"/>
    <add key="ImageUpdateInterval" value="10"/>

    <add key="WebIp" value="127.0.0.1"/>
    <add key="WebPort" value="8088"/>
  </appSettings>
~~~~

The IP and port at which the dashboard can receive performance counter
data is set at the "endpoints" section. Make sure that the ports
correspond to the ones you configured for the Counter Publisher. You can
also add multiple entries here, e.g. if you want to receive data on
several ports. An IP of "0.0.0.0" means that the Dashboard is listening
on all available network interfaces.

~~~~ {.code}

    <endpoints>
      <add  address="0.0.0.0" port="40001" protocol="udp"/>

      <add  address="0.0.0.0" port="40501" protocol="udp"/>
    </endpoints>
~~~~

### Troubleshooting

If graphs are not created or displayed as expected, check the following:

-   Is the Counter Publisher application enabled? Check the
    configuration at
    /deploy/CounterPublisher/bin/CounterPublisher.dll.config.
-   Are Counter Publisher and Dashboard configured to use the same IP,
    port and protocol for sending and receiving counter data?
-   Make sure that the appropriate ports are open in the firewall, if
    Counter Publisher and Dashboard are not running on the same server.
-   Make sure that the Dashboard has write access to
    /deploy/bin\_tools/dashboard/Images and
    /deploy/bin\_tools/dashboard/PerfData.
-   Check the log file at /deploy/bin\_tools/dashboard/log for errors.

