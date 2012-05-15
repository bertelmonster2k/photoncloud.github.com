---
layout: article
title: Interest Management
categories: [photon-server, references]
tags: [overview, mmo]
---
{% include globals %}

MMO Basics
----------

### Interest Management

An object that moves through a virtual world generates position update
events. To display a fluid movement on other clients a minimum of about
10 events per second is required. Clients that are connected to a world
with hundreds or thousands other players can neither receive nor process
all these events due to the limit of resources. Managing the interest is
the key to success: Each client should only get what he needs (what he
sees).

A common solution is region-based interest management: The world is
divided into multiple regions and clients receive the events that
originate from regions within their view distance only.

![](../img/mmo-TR454pic.png) Regions of interest around a ship

[http://staff.cs.utu.fi/\~jounsmed/papers/TR454.pdf](http://staff.cs.utu.fi/~jounsmed/papers/TR454.pdf)
