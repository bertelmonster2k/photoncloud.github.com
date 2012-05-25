---
layout: article
title: Cached Events
categories: [photon-cloud, photon-server, references]
tags: [lite, lobby, how-to, overview]
---
{% include globals %}

## Overview

Usually, the operation RaiseEvent sends your event to anyone else,
currently in the same room. Cached events are ... cached ... and will be
sent to joining (new) players when they arrive.

Use case: Position updates are sent 10 times a second, so joining
players will get your position quickly. Your player's nickname, color
and clothing don't need ten updates per second. Once sent, it would be
nice to "remember" and update new players when they arrive. This is what
cached events do.

Cached events can be modified and removed as needed. Also, they are
sorted for delivery. See below.

#### Disclaimer

Cached events are a feature of the Lite and Lite Lobby applications of
Photon 3. They are not supported by the MMO application or if you start
from scratch.

## Cache control

Events are cached per actor and per event-code. This way, every player
can send event "MyProfile" (a.k.a. (byte)0) and set some values.

Per event, the cache can be updated and replaced or simply removed.

Updating an event is done by merging new content into the previously
cached one. The newer event's Data Hashtable will update (key by key)
any existing value.

Photon 3 allows you to send Null as value in Hashtables (and several
other places), so to get rid of a key in a cached event: Send the same
event, assign null to the key you want deleted and merge the event.

## Ordered delivery

Cached events are sent to players when they join, which means they make
up the fist few events they get inside a room.

New players get cached events, grouped by player, sorted by ascending
actor number. This gives you all cached events of player 1 before you
get other player's events.

Per (sending) player, cached events are ordered by ascending eventCode.
No matter if event 10 was sent before event 1, if it's coming from the
cache, event 1 will precede event 10 (and 2 and so on).

This should give you all control about the order you need. You can even
cache stuff like "map" or "match rules" this way: put them in the event
with lowest number and new players know this first.
