---
layout: article
title: Performance Tips
categories: [photon-server, photon-cloud, references]
tags: [setup, how-to, overview]
---
{% include globals %}

Perfomance Tips
---------------

### Call Service regularly

The client libraries rely on regular calls to LitePeer.Service, to keep
in touch with the server. Bigger pauses between the Service calls could
lead to a timeout disconnect, as the client can't keep up the
connection.

Loading data is a common situation where less updates per second are
done in the main loop. Make sure that Service is called despite loading
or the connection might suffer and be closed. If overlooked, this
problem is hard to identify and reproduce.

### Updates vs. Bandwidth

Ramping up the number of updates per second makes a game more fluid and
up-to-date. On the other hand, used bandwidth might increase
dramatically. Keep in mind that possibly each Operation you call will
create events for other players.

On a mobile client 4 six 6 operations per second are fine. In some cases
even 3G devices use pretty slow networking implementations. Keep in mind
that it might in fact be faster to send fewer updates per second.

PC based clients can go a lot higher. The target framerate should be the
limit for these clients.

### Producing and consuming data

Related to the bandwidth-topic is the problem of producing only the
amount of data that can be consumed on the receiving end. If performance
or framerate don't keep up with incoming events, they are outdated
before they are executed.

In the worst case, one side produces so much data that it breaks the
receiving end. While developing, keep an eye on the queue length of your
clients.

### Datagram size

The content size of datagrams is limited to 1200bytes to run on all
devices

Operations and events that are bigger than 1200bytes get fragmented and
are sent in multiple commands. These become reliable automatically, so
the receiving side can reassemble and dispatch those bigger data chunks
when completed.

Bigger data "streams" can considerably affect latency as they need to be
reassembled from many packages before they are dispatched. They can be
sent in a separate channel, so they don't affect the "throw away"
position updates of a (lower) channel numnber.
