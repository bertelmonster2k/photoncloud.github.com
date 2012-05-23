---
layout: article
title: Unity Island Demo
categories: [photon-server, references]
tags: [sample, unity, demo, mmo]
---
{% include globals %}

## MMO Island Demo

The MMO Island Demo is a visually rich, 3d demo client that uses the MMO
Application. You can either run it many times to get an impression of
the interest management done or you can use it cooperatively with the
"Win Grid" client described in it's own page.

Before you try to run any of the clients, make sure to start the Photon
Server.

#### Island Setup

The MMO Island Demo is a refactored Island Demo of Unity v2.5. This is a
separate [download from the Unity resources
page](http://unity3d.com/support/resources/example-projects/islanddemo).

Our part of the Island client can be found in the Server SDK. Navigate
to: \\src-server\\Mmo\\Photon.MmoDemo.Client.UnityIsland. The folders
from the Island.zip must be merged into that folder but *don't replace
any files*. The readme.txt provides help in case of issues.

Open the scene Islands.unity in the Assets folder. Unity will upgrade
the scene if needed but might fail doing so. If the process crashes, try
again! Unity usually recovers after an initial issue. When the Editor is
open, the compiler might list two errors (open the console view).
Doubleclick to open the source and simply comment out both lines.

After this is done, you can start the scene in the Editor and run around
the island but you are alone. The Island client does not provide bots or
multiple connections.

#### More players

To simulate more players on a single machine, use [the WinGrid
Demo](/democlientwingrid). It is also in the Server SDK but must be
built first.

Open the solution \\src-server\\Mmo\\Photon.Mmo.sln in Visual Studio or
Mono Develop. Select the project "Photon.MmoDemo.Client.WinGrid" to
build and run it.

A new window shows the heightmap of the island and you can add bots by
pressing the Insert key. They should turn up in the Island Demo in Unity
on the radar and you can adjust the view-area with the mouse wheel.
