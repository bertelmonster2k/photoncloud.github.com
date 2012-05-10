---
layout: article
title: Setup and Config
categories: [photon-server, getting_started]
tags: [how-to, getting_started, setup, quickstart, installation]
---
{% include globals %}

Setup
-----

This chapter explains how files and folders are organized for the Photon
Server and how things are setup. Everything needed is in the deploy
folder.

### Organization of Server And Apps

There are four versions of Photon in the folders: "bin\_Win32",
"bin\_x64", "bin\_Win32\_xp" and "bin\_win64\_xp". We refer to these as
binaries-folders.

Per application, Photon requires a separate folder next to the
binaries-folder (e.g. "Lite" in the deploy folder). The assemblies must
be in a "bin" subfolder (e.g. Lite/bin).

![](../img/Folder-Structure-Applications.jpg) Folder Structure for
Deploy

The following folders in the Server SDK deploy folder are applications:
Lite, LiteLobby, MMO, CounterPublisher, Policy

Applications are setup in Photon's config file, as explained below.

The bin\_tools folder currently contains useful tools:

-   **Baretail:** Our favourite log viewer in the free edition. It is
    used by PhotonControl to view the latest logs.
-   **Photon Dashboard:** The service to collect and concentrate
    Dashboard Counters and display them on a webpage.
-   **Perfmon:** Contains a list of PerfMon counters used when you setup
    counter logging to a file. This is explained under "Administration".
-   **Stardust:** A commandline testclient that can be used to get some
    load on a machine. This is shown in "Photon in 5 Minutes"

### Configuration: PhotonServer.config

The main configuration file for Photon is the PhotonServer.config. An
identical copy is located in each binaries-folder in the SDK. It is used
to setup applications, listeners for IPs and performance specific
values. It does not contain config values for the game logic.

The default values make sure Photon scales nicely on more cores but does
not overwhelm a regular machine. In general, performance tweaks are not
needed.

The following settings are most commonly used. More options are
described in the "photon-configuration.pdf", located in the Server SDK.

#### The Application Node

The config file defines which applications Photon should load on
startup. In the "Applications" node, several "Application" entries can
be added.

~~~~ {.code}
    
        <Applications Default="Lite">
            <!-- Lite Application -->
            <Application
                Name="Lite"
                BaseDirectory="Lite\Lite"
                Assembly="Lite"
                Type="Lite.LiteApplication"
                EnableAutoRestart="true"
                WatchFiles="dll;config"
                ExcludeFiles="log4net.config">
            </Application>
        </Applications>
    
~~~~

Each application is assigned a name by which the clients reference it on
connect (compare: [Photon Client Workflow](/photonclient/overview)).

The "BaseDirectory" defines the folder in which an application resides.
It does not name the "\\bin" folder but expects it by convention. The
"Assembly" names the .dll of an application and "Type" its "main" class
(which derives from Photon.SocketServer.Application).

The setting "EnableAutoRestart" tells Photon to start a shadow copy of
the loaded application. This allows developers to replace the files
without locking issues, making development more convenient. Photon will
automatically start a new instance of an application, 10 seconds after
files are changed. "WatchFiles" and "ExcludeFiles" refines the list of
files that trigger a restart. Clients that connected to a previous
shadow copy will stay in it. New connects will use the new logic.

To load multiple applications, simply add more Application nodes (with
unique name). One of them can be made the default application by using
it's name ("Lite" in the example). The default app is the fallback for
clients that try to connect to an unknown application.

Applications that are not setup in the PhotonServer.config won't be
loaded and don't need to be deployed. If an application is in the config
but the files are missing, Photon won't start and name the issue in the
logs.

#### UDPListeners and TCPListeners Nodes

These configure UDP and TCP endpoints on your machine respectively. You
can use either (e.g. only UDP) or both.

The default IP 0.0.0.0 makes Phonton listen on any locally available IP.
By replacing the wildcard IP, Photon will open only specific IPs and
ports. Multiple UDPListener and TCPListener nodes can be defined,
opening several IP/port combinations.

Per UDPListener and TCPListener node, you can setup an
OverrideApplication or DefaultApplication. Override means: any client
that connects to this port will end up in the application named, no
matter what the client connects to. Default is a fallback, in case the
application named by a client is not found.

~~~~ {.code}
    
        <UDPListeners>
            <UDPListener
                IPAddress="0.0.0.0"
                Port="5055"
                OverrideApplication="Master">
            </UDPListener>
            <UDPListener
                IPAddress="0.0.0.0"
                Port="5056"
                OverrideApplication="Game1">
            </UDPListener>
        </UDPListeners>
    
~~~~

Note: If you are using a license that's bound to an IP, you need to set
this IP in the config.

#### TCPSilverlightListeners and TCPFlashListeners Nodes

Could be removed but are needed when you create Silverlight or Flash
games respectively. Both client-side plugins require a server to respond
with a "policy file" (unless the website is on the same domain as
Photon).

#### Timeout Settings

Two values in the instance node describe how the server times out
unresponsive UDP clients: MinimumTimeout and MaximumTimeout.

~~~~ {.code}
    
        <Instance1
            EnablePerformanceCounters = "true"
            DataSendingDelayMilliseconds="50"
            AckSendingDelayMilliseconds="50"
            MinimumTimeout="5000"
            MaximumTimeout="30000">
    
~~~~

A peer connected with UDP has MinimumTimeout milliseconds to respond
before it can be disconnected. The actual time until disconnect, is
determined dynamically per peer based on the RTT history. Previously
good RTTs will disconnect faster.

TCP connections have a separate "InactivityTimeout" setting in the nodes
TCPListener and TCPPolicyListener (also in the PhotonServer.config). We
set those to 5 seconds (5000ms). If no request arrives in this time, the
connection will be timed out and closed. Clients will also regularly
Ping the server, to avoid this. Clients don't timeout for TCP.

Keep in mind that a client independently monitors a connection and might
time it out as well. Both sides should have similar timeouts, fitting
your game.

#### Send Delay and Ack Delay

The attributes DataSendingDelayMilliseconds and
AckSendingDelayMilliseconds represent a tradeoff between performance and
minimal response times. This delay directly adds some lag to reduce the
bandwidth usage: the wait allows the server to aggregate commands and
send them in one package. The "send" delay is triggered when the server
sends anything, the "ack" delay by incomging reliable data.

As you can see above, the default values are 50ms each. We found this to
be a good value but it causes a \~50ms roundtrip time, even if client
and server run on the same machine.

Depending on your game, you should load test with different values. A
delay of 0 is a special case that skips usage of timers, so avoid low
delays < 10.
