---
layout: article_server
title: Starting Photon in 5 Minutes
categories: [photon-server, getting_started]
tags: [how-to, getting_started, setup, quickstart, installation]
---

Photon is extremely easy to install and start.
The SDK includes ready-to-use binaries which can be up and running within 5 minutes.

The following video shows how this is done. Below that, the essential steps are also available to read.

<object width="640" height="385">
    <param name="movie" value="http://www.youtube.com/v/hu7D5oc0Ags?fs=1&amp;hl=en_GB&amp;rel=0"></param><param name="allowFullScreen" value="true"></param>
    <param name="allowscriptaccess" value="always"></param>
    
    <embed src="http://www.youtube.com/v/hu7D5oc0Ags?fs=1&amp;hl=en_GB&amp;rel=0" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" width="640" height="385"></embed>
</object>


###Download

The Photon Server SDKs are available on our website:
[www.exitgames.com/Download/Photon](http://www.exitgames.com/Download/Photon)

Everything you need is in the "Stable" section. Choose the "ExitGames-Photon-Server-SDK" for now.


### Choose Fitting Binaries
Unzip the files anywhere on your machine. Locate the "deploy" folder and chose the fitting binaries for your system:

- **bin_Win32**: 32 bit Windows Vista and up
- **bin_x64**: 64 bit Windows Vista and up
- **bin_Win32_xp**: 32 bit Windows XP or 2003
- **bin_Win64_xp**: 64 bit Windows XP or 2003

The deploy folder of the SDK contains everything you need to run Photon. All other content of the SDK is documentation, libraries and source.


### Photon Control

Run Photon's admin tool: PhotonControl.exe. This will create a traybar icon and menu to manage Photon.

<figure>
<img src="{{ IMG }}/PhotonControl-Photon-RunApp.jpg" />
<figcaption>"Run as application" in Photon Control</figcaption>
</figure>

Click the white/blue icon and from the menu, select "Photon", "Start as Application". Now you started Photon!

It might take several seconds until Photon is ready to use, depending on the applications it runs.


### Start the Testclient

The server SDK includes a testclient to simulate multiple clients and get some load.
You can start it from the Photon Control menu: Photon, Run Testclient.

The Testclient is a simple console application which will simulate 100 in 25 games with 4 players each.

<figure>
<img src="{{ IMG }}/PhotonControl-Photon-TestClient.jpg" />
<figcaption>Successfully connected Testclient</figcaption>
</figure>

Alternatively you can download the Photon Client SDK for DotNet.
In it, the "Realtime Demo" is provided as compiled executable, configured for "localhost".
Start it twice, so each gets updates of the other.
