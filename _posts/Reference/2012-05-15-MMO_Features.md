---
layout: article
title: MMO Features
categories: [photon-server, references]
tags: [overview, quickstart, mmo]
---
{% include globals %}

MMO Features
------------

-   Region-based interest management
    -   Square tile algorithm with variable width, height and tile size
        is implemented.
    -   Easily replaceable with other region algorithms (hexagonal, etc
        – see
        [http://gram.cs.mcgill.ca/papers/boulanger-06-comparing.pdf](http://gram.cs.mcgill.ca/papers/boulanger-06-comparing.pdf))

-   Items (Avatars, NPCs, shared game objects)
    -   Clients can spawn, destroy and move items.
    -   Items can have properties that are readable by other clients and
        changeable by the owner.
    -   Item properties have a revision number: Clients that lose sight
        of an item for some time can compare the revision to determine
        if they need to receive updated properties.

-   Interest areas: Events of items within an interest area are
    automatically received.
    -   Interest areas have two interest thresholds. Items that enter
        the inner radius become visible; items that leave the outer
        become invisible. This optimization reduces frequent visibility
        changes. \
        ![](../img/mmo-feature1.png)Inner and outer radius
        -   invisible item, out of range
        -   invisible item enters outer interest area
        -   invisible item enters inner interest area and becomes
            visible
        -   visible item leaves inner interest area
        -   visible item leaves outer interest area and becomes
            invisible

    -   Interest areas can be resized: Adjusting the view distance in
        proportion to the amount of seen items can be beneficial
        -   To improve the performance in crowded areas or
        -   To show distant items in areas with low population.

    -   Interest areas can be attached to any item: Whenever the item
        moves the interest area follows and changes the interest
        accordingly. This is especially useful to move the interest area
        with the own avatar.
    -   Detached interest areas can be moved freely. This is beneficial
        for camera flights.
    -   Clients can have multiple interest areas to show different parts
        of the world at the same time.

-   Manual interest management: Clients can manually (un-)declare
    interest in items.
-   Custom events can be sent through items to two possible targets:
    -   The item owner
    -   All clients that are interested in the item (item subscriber)

-   Optimized position updates: Clients send updates only when they
    move.
-   Duplicate user recognition: A subsequent user connection resets the
    previous connection.

