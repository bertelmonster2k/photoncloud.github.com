---
layout: article
title: Base Applications
categories: [photon-server, getting_started]
tags: [how-to, getting_started, setup, quickstart, installation]
---
{% include globals %}

Base Applications
-----------------

The Photon Server SDK includes several applications that should provide
a good starting point for your own development. Below is described what
each does and for which game-style it's a useful basis.

#### Lite

The Lite Application is probably referenced most often. It implements
the basic framework for room based games in which players get together
to play a match in a small group. Each room reflects a game and is
identified by its name. Anyone connected can join and leave without
restriction. Inside a room, any client can raise events to send data
across to the others. Properties can be set for rooms and actors to keep
certain values available for everyone joining.

Games which don't need an authoritative server can be built on top of
Lite without modifications.

This application can easily be extended with server-side logic per room.
Matchmaking could group players in rooms, depending on your game's
requirements.

[Read more: Lite and Lite Lobby](/liteandlitelobbyaddon)

#### Lite Lobby

The Lite Lobby is an example of how to build on top of Lite and extend
it's functionality. Lite Lobby adds new types of rooms: Lobbies. These
won't send join/leave event but list currently used room names. The
player (or client code) can easily chose from running games.
Additionally, the count of users in a room is provided.

This application adds "manual matchmaking" to Lite and could also be a
basis for your game.

[Read more: Lite and Lite Lobby](/liteandlitelobbyaddon)

#### MMO

The MMO Demo Application is a good solution for games where all players
share a large world. It provides interest management and most common
base classes for items, actors, properties and more.

Typically games with a shared world contain more game-specific logic on
the server side, so this application should be understand as a
framework.

[Read more: MMO](/mmo)

#### Policy

The Policy Application runs on Photon to send the crossdomain.xml.
Webplayer platforms like Unity Webplayer, Flash and Silverlight request
authorization before they contact a server.

It has it's own page in the DevNet: [Policy
Application](/PolicyApp)

#### CounterPublisher

This application reads and forwards local PerfMon Counters of Photon
Core to the Dashboard. It does not interfere with other applications and
client should not connect to it.
