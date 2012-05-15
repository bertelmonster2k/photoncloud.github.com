---
layout: article
title: Binary Protocol
categories: [photon-server, photon-cloud, references]
tags: [overview, how-to]
---
{% include globals %}

Binary Protocol
---------------

Photon and its clients are using a highly optimized binary protocol to
communicate. It's compact, yet easy to parse. You don't have to know its
details, as it is handled behind the scenes. Just in case you want to
know it anyways, this page describes it.

Communication Layers
--------------------

The Photon binary protocols are organized in several layers. On its
lowest level, UDP is used to carry the messages you want to send. Within
those standard datagrams, several headers reflect the properties you
expect of Photon: optional reliability, sequencing, aggregation of
messages, time synchronization and various others.

The following chart shows the individual layers and their nesting:

![](../img/BinaryProtocol-udp-layers.png) Layers of the binary protocol

In words: Any UDP package contains an Enet header and at least one enet
command. Each enet command carries our messages: An Operation, result or
event. Those in turn consist of the operation header and the data you
provided.

Below, you will find tables listing each part of the protocols and it's
size. This should give you an idea of the bandwidth needed by certain
content you send. If you wanted to, you could calculate how big any
given message becomes.

To save you this work, we have an example, too.

Example: Join "somegame"
------------------------

Let's take a look at the operation join (implemented by the Lite
Application). Without properties, this is an operation with a single
parameter.

The "operation parameters" require: 2 + count(parameter-keys) +
size(parameter-values) bytes. The string "somegame" uses 8 bytes in UTF8
encoding and strings require another 3 bytes (length and type
information). **Sum: 14 bytes.**

The parameters are wrapped into an operation-header, which is 8 bytes.
**Sum: 22 bytes.**

The operation code is not encoded as parameter. Instead it is in the
operation-header.

The operation and its header are wrapped into Enet's "send reliable"
command as payload. The command is 12 bytes. **Sum: 34 bytes.**

Let us assume the command is sent when no other commands are queued. The
Enet package header is 12 bytes. **Sum: 46 bytes for the reliable,
sequenced operation.**

Last but not least, the Enet package is put into a UDP/IP datagram,
which adds 28 bytes headers. Quite a bit, compared to the 46 bytes so
far. Sadly, this can't be avoided completely but our aggregation of
commands can save you a lot of bandwidth, sharing those headers.

**The complete operation takes up 74 bytes.**

The server will have to acknowledge this command. The ACK command is 20
bytes. If this is sent alone in a package, it will take up 40 bytes in
return.

![](../img/BinaryProtocol-HexBytes.jpg) Join "somegame" in Wireshark

Operation Content - Serializable Types
--------------------------------------

type (C\#)

size [bytes]

description

byte \

2 \

8 bit \

boolean \

2 \

true or false \

short \

3 \

16 bit \

int \

5 \

32 bit \

long \

9 \

64 bit \

float \

5 \

32 bit \

double \

9 \

64 bit \

string \

3 + size( UTF8.GetBytes(string) ) \

< short.MaxValue length

byte-array \

5 + 1 \* length \

< int.MaxValue length

int-array \

5 + 4 \* length \

< int.MaxValue length

array of <type\> \

4 + size(entries) - count(entries) \

< short.MaxValue length

hashtable \

3 + size(keys) + size(values) \

< short.MaxValue pairs

operation parameters \
 \

2 + count(parameter-keys) + size(parameter-values) \

< short.MaxValue pairs\
 special case of a hashtable - see below

On the client side, operation parameters and their values are aggregated
within a Hashtable. As each parameter is resembled by a byte key,
operation requests are streamlined and use less bytes than any "regular"
hashtable.

Results for operations and events are encoded in the same way as
operation parameters. An operation result will always contain: opCode,
returnCode and a debug string. Events always contain: evCode, actorNr
and the event data Hashtable.

Enet Commands
-------------

name

size

sent by

description

connect \

44 \

client \

reliable, per connection\

verify connect \

44 \

server \

reliable, per connect\

init message \

57 \

client \

reliable, per connection (choses the application)\

init response \

19 \

server \

reliable, per init\

ping \

12 \

both \

reliable, called in intervals (if nothing else was reliable)\

fetch timestamp \

12 \

client \

a ping which is immediately answered \

ack \

20 \

both \

unreliable, per reliable command\

disconnect \

12 \

both \

reliable, might be unreliable in case of timeout\

send reliable \

12 + payload \

both \

reliable, carries a operation, response or event\

send unreliable \

16 + payload \

both \

unreliable, carries a operation, response or event\

fragment \

32 + payload \

both \

reliable, used if the payload does not fit into a single datagram\

UDP Packet Content - Headers
----------------------------

name

size [bytes]

description

udp/ip \

28 + size(optional headers) \

IP + UDP Header. \
 Optional headers are less than 40 bytes. \

enet packet header \

12 \

contains up to byte.MaxValue commands \
 (see above) \

operation header \

8 \

in a command (reliable, unreliable or fragment) \
 contains operation parameters \
 (see above) \

