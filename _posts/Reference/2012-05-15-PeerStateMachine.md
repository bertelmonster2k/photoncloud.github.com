---
layout: article
title: Peer State Machine
categories: [photon-server, references]
tags: [mmo, overview]
---
{% include globals %}

MMO Peer State Machine
----------------------

A client peer has one of the following states:

1.  Connected: Initial state after connecting, player has not entered a
    world.
2.  WorldEntered: Player entered a world with operation EnterWorld.
3.  Disconnected: Client disconnected.

This diagram shows all transitions: \

![](../img/mmo-stateMachine.png)MMO Peer State Machine

The peer uses a different operation handler for each state. \
 This allows operations to behave differently in each state. \
 The following operation handlers are used:

1.  State Connected: Photon.MmoDemo.Server.MmoPeer
2.  State WorldEntered: Photon.MmoDemo.Server.MmoActor
3.  State Disconnected:
    Photon.SocketServer.Rpc.OperationHandlerDisconnected

The client should always wait for the related operation response / event
after calling a state changing operation; the peer state and therefore
the current operation handler (operation behavior) is not guaranteed
until the response / event is received. For operation *EnterWorld* it's
the operation response, for operation *ExitWorld* it's the event
*WorldExited*.
