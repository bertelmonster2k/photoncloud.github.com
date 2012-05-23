---
layout: article
title: Photon Control Application
categories: [photon-server, getting_started]
tags: [setup, installation]
---
{% include globals %}

##Administration

### Photon Control

The PhotonControl.exe is a small tray application that manages Photon
for you. It runs as separate process and shows as tray icon with
right-click menu. A gray icon shows that Photon is not running. When it
spots a Photon process running, it turns blue.

You can setup PhotonControl to autorun on login, making it easier to
spot Photon on any machine.

<figure>
<img src="{{ IMG }}/PhotonControl-EnableAutostart.jpg" />
<figcaption>Enable PhotonControl Autostart</figcaption>
</figure>

The key features are:

-   Photon Control reads the license file on start and displays key
    values.
-   Start / Stop Photon as application or install and control it as
    service instead.
-   Install / remove Photon's PerfMon counters, start Windows PerfMon or
    log counters to a file.
-   Explore working path to quickly find the place where Photon is
    running and where the logs are.

Photon Control needs to be run with Admin rights to install/remove
services and the PerfMon Counters.

#### Running Photon

On a public server, Photon should run as service. This requires only
three steps in PhotonControl:

-   Install the Photon service
-   Install the PerMon Counters (they might prove useful)
-   Start Photon as service

The PerfMon Counters are not exactly a requirement to run but they can't
be enabled while Photon is running. Using the Dashboard Counters is
explained in a separate Chapter.

Running Photon as application is an alternative for local development.

**Before you move Photon** from one folder to another, you should make
sure to remove all services and also PhotonControl autostart. Install
again at the new location.

### Logs

Photon is logging essential information like it's state and exceptions to
several log files. There are two "log" folders:

-   The deploy/log folder is created after start for application logs.
    Anything that is logged by your business logic goes into a fitting
    file in deploy/log.
-   The other log folder is where your running executable is (e.g.
    bin\_win32/log). This is where you find logs of Photon's Core.

We used log4net in the logic layer, which can be extensively configured
and very useful. [Learn more about Log4Net
here](http://logging.apache.org/log4net/).

### Counters

Photon keeps track of several essential values as support for
performance- and error-analysis. These are published in two separate
sets of Counters: "PerfMon Counters" and "Dashboard Counters".

#### PerfMon Counters

The PerfMon Counters are key values from the Photon Core. They track
values like connected peers, package count, reliable UDP usage, bandwith
and much more. These Counters cannot be changed by the developer and are
accessed by PerfMon. Perfmon is a GUI to create those performance graphs
and is already installed with Windows.

If PerfMon Counters should be logged over a longer time (and without
running the gui), PhotonControl can setup and start logging to a file.
Under "PerfMon Counters", click "Create Logging Set" and "Start
Logging". The logs should end up under
C:\\PerfLogs\\Admin\\photon\_perf\_log\_<date\>.blg

#### Dashboard Counters

The Dashboard Counters track values within the business logic and can be
extended as needed.

### Dashboard

The Dashboard is a service which aggregates Dashboard Counters and
generates graphs on a website to monitor them.

The Dashboard itself can be installed as service while Photon is already
running. Which Counter Data an Application publishes (if at all),
depends on the Application's setup. Lite has several pre-defined
Counters ready to use.

### Troubleshooting

If Photon does not behave as expected, always have a look at the logs.

These are the most common pitfalls you should check as well.

-   **Missing DotNet 3.5:** In this case, Photon can't start. Don't mix
    this up with the CLR Version. The 2.0 CLR version is also used by
    DotNet 3.5.
-   **Application missing:** If Photon does not find one of the
    configured applications, it can't start. Check the configuration
    against deployed folders.
-   **Build not up to date:** Switching to a new server SDK, you should
    always re-build your applications, referencing the assemblies from
    the libs folder. Otherwise they might be incompatible.
-   **Firewall:** If Photon is running but not accessible from another
    machine, check the Firewalls. Newer Windows versions have roles and
    rights and your hoster most likely used hardware firewalls.
-   **Lag:** By default, the roundtrip time is about 50ms, even locally.
    This depends on a setting, explained on the setup page. Read: [Send
    Delay and Ack Delay](/setupandconfig)

### Dump File Setup

If a server crashes and the reason for it is not found in the logs,
Photon can be configured to create dump files. These reflect the state
and memory of the crash and are valuable to debug these cases.

To enable the feature, you need to edit your PhotonServer.config. Set
the instance-attribute "ProduceDumps" to true and restart the server. It
might look like this example:

~~~~ {.code}
    
        <Instance1
            EnablePerformanceCounters = "true"
            DataSendingDelayMilliseconds="50"
            AckSendingDelayMilliseconds="50"
            MinimumTimeout="5000"
            MaximumTimeout="30000"
            ProduceDumps="true">
    
~~~~

This will write up to 10 "full" dump files. Once a dump file is written,
you can zip it with the logs and mail it to us with a description of the
issue. In most cases, we will get in touch with you to get more
information and solve the case.
