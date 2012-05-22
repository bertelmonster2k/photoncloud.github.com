---
layout: article
title: Receiver Groups
categories: [photon-cloud, photon-server, references]
tags: [lite, lobby, how-to, overview]
---
{% include globals %}


Receiver groups are an alternative way to define who receives an event.
It's an option available with Lite's operation RaiseEvent.

Usually, Lite sends your events to everyone else in the same room. With
the receiver group parameter for RaiseEvent, you can send an event to
"All", including yourself. If needed, you can also send to only the
"MasterClient".

This is an alternative to listing all target players, which is still
possible in RaiseEvent.

### MasterClient

The MasterClient is the one with the lowest actorNumber in the room and
thus the one who's there the longest time.

This doesn't give the MasterClient any privileges. But it could.
Example: The MasterClient is always the one to start a new round of your
game (in the same room).

If the current MasterClient leaves, a new one is assigned immediately.
This is done by convention, not explicitly by events.

#### Disclaimer

Receiver groups are a feature of the Lite and Lite Lobby applications of
Photon 3. They are not supported by the MMO application or if you start
from scratch.
