---
layout: article
title: First Steps
categories: [photon-cloud, getting_started]
tags: [overview, quickstart, how-to, setup, installation]
---
{% include globals %}

### Getting an Account

The Unity package "Photon Unity Networking" includes a setup wizard,
which opens automatically when you import the plugin. Simply submit your
email address in the Photon Unity Networking window to generate your
account and Cloud App ID. It is effective immediately. Done.

If your email is already connected with an account, you can visit out
webpage to get your Cloud App ID. We will mail you the credentials to
login, to set a password and to access your Cloud App ID if needed.

Registration is also possible on the webpage:
[cloud.exitgames.com/Account/LogOn](http://cloud.exitgames.com/Account/LogOn).

### Photon Cloud App ID

The Cloud App ID is a generated identifier for your Photon Cloud
application. It's used when an application client connects and separates
your users from anyone else's.

### Next Steps

Once you have your own Cloud App ID, you should use it within the
provided samples. Check out how the GUI creates a room, which states
happen when the client "moves" from the Master Server to a Game Server
and how RPCs are used to exchange data.

### Application logic

Currently, you can not modify the server's logic within the cloud. The
flexible "Loadbalancing" framework should enable you to build any
room-based realtime (and turnbased) game. The server simply forwards all
RPCs and synchronization events between your clients.

### Self hosting

As alternative to the Photon Cloud Service, you can download the [Photon
Server SDK](http://www.exitgames.com/Download/Photon). It includes the
binaries to run a server on suitable Windows machines and the source to
modify the server's logic.

Switching to your own server should be simple, as the basic workflow and
protocol stays intact. The Cloud App ID is no longer used and you need
your own Photon license. Also, you have to setup your server machine.
