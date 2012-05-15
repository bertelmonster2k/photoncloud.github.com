---
layout: article
title: Boot Camp Demo: Objective
categories: [photon-cloud, tutorials]
tags: [sample, how-to, unity, demo]
---
{% include globals %}

Bootcamp Multiplayer Demo
=========================

![](../img/Demo-BootcampMultiplayer.png) Bootcamp Multiplayer Demo

The Demo is available for download at the **[Unity Asset
Store](http://u3d.as/content/exit-games/photon-bootcamp-demo/1AA)**.

Thanks for coding the demo and for writing this tutorial to:\
\
 **[King's Mound](http://www.kingsmound.com)**\
 Mina 1014, Barrio Antiguo, Monterrey, Mexico\
 T. + 52 818 340 4826\
 [www.kingsmound.com](http://www.kingsmound.com)

Objective
---------

Describing the actions needed to add multiplayer functionality to the
Bootcamp Demo with Exit Games Photon.

The first thing we did was going through the game and making a list of
all actions that need to be synchronized with other players. For
Bootcamp, we came up with the following list:

1.  Position of Player
2.  Rotation of Player
3.  Animations of Player in their different States
4.  Action: Firing
5.  Action: Reloading
6.  Action: Crouching
7.  Action: Aiming (including torso rotations)
8.  Current Selected Weapon
9.  Player Health
10. Player Death
11. Chat Messages
12. Joining the Lobby

Few initial considerations: The server for this demo will not be
authoritative, the shooting player determines if he hits another player
or not. This is something we definitely want to rethink for future
implementations to avoid cheating. Crouch and Aim are treated
differently than motion actions because they can be blended with them.

To handle all animation variants to be executed without sending a
particular message for each animation, the following scripts were
created for the local player:


