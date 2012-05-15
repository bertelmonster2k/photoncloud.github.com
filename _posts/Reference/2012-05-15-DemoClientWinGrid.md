---
layout: article
title: WinGrid Demo
categories: [photon-server, references]
tags: [mmo, how-to, sample, demo, ]
---
{% include globals %}

MMO WinGrid Demo Client
-----------------------

### Overview

![](../img/mmo-WinGrid1.jpg)WinGrid Demo

-   Each cell is one interest region.
-   The red dot is the avatar item.
-   The red inner square is the inner interest area radius (default:
    size of 1 cell)
-   The red outer square is the outer interest area radius (default:
    size of 3 cells)

Mouse input:

-   Left button drag: Move avatar
-   Wheel: Increase or decrease interest area
-   Middle button click: Reset interest area to default
-   Right click: Toggle interest area avatar attached/detached

Keyboard input:

-   M: Toggle auto-move on/off
-   Num-Pad 5: Move avatar to center
-   WASD and other Num-Pad Keys: Move avatar
-   +: Open new tab and connect new client
-   -: Close tab and disconnect client
-   Insert: Start bot (additional item on same connection that moves
    like an avatar)
-   Delete: Stop bot
-   Space: Go to Settings tab (2nd tab from the left)

Examples
--------

\1. Client with bots

![](../img/mmo-WinGrid2.jpg)Bots as simple way to add items

Bots do not have an interest area.

\2. Multiple Clients

![](../img/mmo-WinGrid3.jpg)Mutliple Clients in huge Interest Area

Other avatars are displayed with interest area. This information is
exchanged via item properties; other clients will always display the
newest chosen interest area size even if they are out of range when this
change happens.\
 Note that detached interest areas are still displayed as if they were
attached: interest areas are not items and cannot be “seen” if moved
around freely.

Settings
--------

![](../img/mmo-WinGrid4.jpg)WinGrid Client Settings

The settings tab is always the second from the far left.

-   Player Text: A text that is shown as avatar name. It’s an item
    property.
-   Player Color: The color of the player’s avatar. This is an item
    property as well.
-   Send movement interval: The interval how often manual position
    updates are sent and/or how often auto movement happens. Note that
    position updates are only sent if the avatar (the item) moves.
-   Send reliable: Enable to test reliable sending of position updates.
-   Auto Move Interval: The time the avatar is moving in one direction.
-   Auto Move Velocity: The distance an avatar moves per movement
    interval (“Send movement interval”.
-   Auto Move: If auto-move is disabled the bots stop moving as well.

Setting changes apply to the tab with the same title only.\
 ‘Space’ is the shortcut to jump between the settings and the game tab.

Radar
-----

The radar shows position changes of all items in the world every few
seconds.\

![](../img/mmo-WinGrid5.jpg)Radar

This feature has been included with the demo to give the user a better
impression about what’s happening in the virtual world. Global radars a
no typical MMO features, that’s why it’s just part of the sample and not
of the interest management framework.

Counter
-------

The server counter tab is updated every 5 seconds.\
 The picture shows the values of one connected client with one moving
player.\
 The received events peek every 5 seconds: They contain just counter
data, no other events are received. \

![](../img/mmo-WinGrid6.jpg)Counters when players are active

If the player stops moving the operations/sec drop to zero:\

![](../img/mmo-WinGrid7.jpg)Counters with less movement
