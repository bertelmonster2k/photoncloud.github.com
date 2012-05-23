---
layout: article
title: Lite Lobby Concepts
categories: [photon-server, references]
tags: [overview, lite, lobby, how-to]
---
{% include globals %}

LiteLobby is an application that (literally) extends the Lite
application to offer a listing of currently used rooms. To achieve this,
it implements two different types of rooms: the LiteLobbyRoom and a
LiteLobbyGame.

By convention, join will put you into a lobby when the given room name
ends on "\_lobby" and into a game room in all other cases. The Operation
join got an optional parameter to name a lobby where the game is listed.
This puts the responsibility on the client side again and you can create
lobbies on the fly.

### Lobby

The lobby rooms (LiteLobbyRoom.cs) are special as they are not used to
play a match with opponents. Instead, they just provide a list of
available rooms (games), so player can chose and join those.

Despite being a room, too, a lobby won't send events on join and leave
of actors. As a result, players inside a lobby don't notice each other
which prevents a flood of join/leave events. There are no operations
aside from leave. As a player leaves rooms when joining another, leave
is not even required.

On join, the complete games list is sent once to get a client up to
date. After that, each player gets the same update event, which is sent
every few seconds. It lists each game by name its current number of
players. The clients keep and update their initial list. Games that are
empty or full, both simply report 0 players as they should no longer be
in the room list.

To send the room list in intervals, the lobby schedules a message for
itself, once the list is sent. This enqueues the event sending in the
regular message passing mechanism, avoiding threading issues.

### Rooms for games

In LiteLobby, regular games can now be connected to a lobby. The lobby
of a room is set when gets created. If the lobby does not exist, it will
be created as well.

Once a room is connected to a lobby, it is responsible to update the
lobby with it's info. The lobby itself, does not mind what's forwarded
as info to the clients. By default, each room only sends its actor count
and will be "full" when 4 players are in a room.

### LiteLobby Config

To avoid incompatibilities, LiteLobby uses its application config file
to define the event types, update interval and the lobby suffix.

Some values can't be edited this way yet: count of users to make a room
"full", event-values and more.

LiteLobby can be running parallel to Lite but usually a game would use
just one of those. The SDKs Photon Config assigns the name LiteLobby
(used by the clients on connect).
