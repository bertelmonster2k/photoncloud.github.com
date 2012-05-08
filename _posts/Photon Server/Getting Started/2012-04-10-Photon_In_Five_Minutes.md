---
layout: post_server
title: Photon in 5 Minutes
categories: [photon-server, getting_started]
tags: [how-to, getting_started, setup, quickstart, installation]
---

<doc><article>
<h2>Starting Photon in 5 Minutes</h2>
<p>Photon is extremely easy to install and start. The SDK includes ready-to-use binaries which can be up and running within 5 minutes.</p>
<p>The following video shows how this is done. Below that, the essential steps are also available to read.</p>
<object width="640" height="385"><param name="movie" value="http://www.youtube.com/v/hu7D5oc0Ags?fs=1&amp;hl=en_GB&amp;rel=0"></param><param name="allowFullScreen" value="true"></param><param name="allowscriptaccess" value="always"></param><embed src="http://www.youtube.com/v/hu7D5oc0Ags?fs=1&amp;hl=en_GB&amp;rel=0" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" width="640" height="385"></embed></object>


<h3>Download</h3>
<p>The Photon Server SDKs are available on our website:<br />
<a href="http://www.exitgames.com/Download/Photon">www.exitgames.com/Download/Photon</a></p>
<p>Everything you need is in the "Stable" section. Choose the "ExitGames-Photon-Server-SDK" for now.</p>

<h3>Choose Fitting Binaries</h3>
<p>Unzip the files anywhere on your machine. Locate the "deploy" folder and chose the fitting binaries for your system:</p>
<ul>
<li><b>bin_Win32</b>: 32 bit Windows Vista and up</li>
<li><b>bin_x64</b>: 64 bit Windows Vista and up</li>
<li><b>bin_Win32_xp</b>: 32 bit Windows XP or 2003</li>
<li><b>bin_Win64_xp</b>: 64 bit Windows XP or 2003</li>
</ul>
<p>The deploy folder of the SDK contains everything you need to run Photon. All other content of the SDK is documentation, libraries and source.</p>


<h3>Photon Control</h3>

<p>Run Photon's admin tool: PhotonControl.exe. This will create a traybar icon and menu to manage Photon.</p>
<figure>
<img src="../img/PhotonControl-Photon-RunApp.jpg"/><figcaption>"Run as application" in Photon Control</figcaption>
</figure>

<p>Click the white/blue icon and from the menu, select "Photon", "Start as Application". Now you started Photon!</p>
<p>It might take several seconds until Photon is ready to use, depending on the applications it runs.</p>

<h3>Start the Testclient</h3>
<p>The server SDK includes a testclient to simulate multiple clients and get some load. You can start it from the Photon Control menu: Photon, Run Testclient.</p>
<p>The Testclient is a simple console application which will simulate 100 in 25 games with 4 players each.</p>
<figure>
<img src="../img/PhotonControl-Photon-TestClient.jpg"/><figcaption>Successfully connected Testclient</figcaption>
</figure>
<p>Alternatively you can download the Photon Client SDK for DotNet. In it, the "Realtime Demo" is provided as compiled executable, configured for "localhost". Start it twice, so each gets updates of the other.</p>

</article></doc>
