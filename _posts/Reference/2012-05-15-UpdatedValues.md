---
layout: article
title: Updated Values
categories: [photon-cloud, photon-server, references]
tags: [overview, how-to]
---
{% include globals %}


Updated:\
Event-, Operation- and Parameter-Codes
--------------------------------------

In Photon 2.x, our demo applications Lite and Lite Lobby used just about
any (more or less random) value for event-, operation- and
parameter-codes. This easily clashed when our projects are used as basis
and get extended.

For Photon 3.x, we came up with this new rule to assign codes: **Our
codes begin with the max value, then go down.** Example: Operation Join
is now code 255 in Lite. If you want to add a new events (codes) to
Lite, start with 0 and increment as you go.

### Map of old and new codes

The following tables include codes from Lite and Lite Lobby
applications. If you use our named constants for these (which you
should), you don't have to know these values.

OperationCode

Old Value

New Value

Join

90

255

Leave

91

254

RaiseEvent

92

253

SetProperties

93

252

GetProperties

94

251

EstablishSecureCommunication

95

(obsolete)

Ping

104

(obsolete)

EventCode

Old Value

New Value

Join

90

255

Leave

91

254

PropertiesChanged

92

253

GameList (LiteLobby)

1

252

GameListUpdate (LiteLobby)

2

251

ParameterCode

Old Value

New Value

ERR

0

(obsolete)

DBG

1

(obsolete)

GameId

4

255

ActorNr

9

254

TargetActorNr

10

253

Actors

11

252

Properties

12

251

Broadcast

13

250

ActorProperties

14

249

GameProperties

15

248

ClientKey

16

(obsolete)

ServerKey

17

(obsolete)

Cache

18

247

ReceiverGroup

19

246

Data

42

245

Code

60

244

Flush

61

243

LobbyId (LiteLobby)

5

242
