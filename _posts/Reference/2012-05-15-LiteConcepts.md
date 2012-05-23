---
layout: article
title: Lite Concepts
categories: [photon-server, tutorials]
tags: [sample, lite, lobby, operations, overview]
---
{% include globals %}


### Peers

Lite is based on the Application framework for Photon and also uses
"Peer" as reference to a connected player. This is wrapped up and
extended in the class LitePeer.

When a client connects to the Lite Application, a new LitePeer is
created in LiteApplication.CreatePeer. Further requests from that client
(e.g. Operations) will now be handled by the corresponding LitePeer.

In Lite, every LitePeer has a single RoomReference as State. Players can
be only in a single room. The State is set by operation Join.

The OnDisconnect method in LitePeer is called when a peer disconnects.
If the peer is still in a room, this gets a message to remove the peer.

### Rooms

In Lite, peers get into rooms to play together, which is why they are
also called games. Rooms are responsible for everything. Everything
aside from join, that is.

Every game has a unique name and is treated as completely separated from
any other. Also, every request that comes from a player is handled in
sequence. Because many rooms exist in parallel and each request is
handled in mere ticks, this setup scales well.

Rooms have lists of players which they update on join, leave or
disconnect. Between these players, events can be sent.

Within a room, a player is also known as Actor and each Actor has an
ActorNumber. That number is used to identify the origin of an event.

### Operations and Events

Lite defines these Operations

-   **Join:** Enter any room by name. It gets created on the fly if it
    does not exist. Join will implicitly leave the previous room of a
    peer (if any). Returns the assigned ActorNumber.
-   **Leave:** Leaves the room (but keeps the connection).
-   **RaiseEvent:** Tells the room to send an event to the other peers
    in it. Events have an EventCode and can carry data provided by the
    client. Lite does not store events.
-   **GetProperties:** In Lite, Properties can be attached to Rooms and
    Players. With this Operation, they are fetched.
-   **SetProperties:** Attaches arbitrary key-value pairs ro a room or a
    single player. Lite does not use the data, so the client can send
    arbitrary data along. Code by convention.

Lite defines these Events

-   **Join:** Joining a room (by Operation) updates the players list
    inside. This is sent in an event to all players (including the new
    one).
-   **Leave:** If a player leaves a room, the others are updated again.
-   **Properties update:** When properties are changed, there is the
    option to broadcast the change. This event takes care that every
    client is up to date.

### RaiseEvent / Custom Events

With Lite, players can send events with arbitrary content to the other
players in a room. The operation RaiseEvent takes care of this and
forwards the event content accordingly.

As an example, this client code from the DotNet Realtime Demo sends a
player's position:

~~~~ {.code}
    
        // Raises an event with the position data of this player.
        internal void SendEvMove(LitePeer peer)
        {
            if (peer == null)
            {
                return;
            }

            Hashtable evInfo = new Hashtable();

            evInfo.Add((byte)STATUS_PLAYER_POS_X, (byte)this.posX);
            evInfo.Add((byte)STATUS_PLAYER_POS_Y, (byte)this.posY);
            //OpRaiseEvent(byte eventCode, Hashtable evData, bool sendReliable, byte channelId, bool encrypt)
            peer.OpRaiseEvent(EV_MOVE, evInfo, isSendReliable, (byte)0, Game.RaiseEncrypted);
        }
    
~~~~

The Lite Application does not try to interpret your events server-side,
so you can send justabout anything you want to. The EV\_MOVE above is
nothing Lite knows, it's only defined and used client-side.

The move event that's raised in the sample above causes a EventAction
call on the receiving clients. They handle the event and data like this:

~~~~ {.code}
    
        //in Game.cs:
        public void EventAction(byte eventCode, Hashtable photonEvent)
        {
            int actorNr = 0;
            if (photonEvent.ContainsKey((byte)LiteEventKey.ActorNr))
            {
                actorNr = (int) photonEvent[(byte) LiteEventKey.ActorNr];
            }

            // get the player that raised this event
            Player p;
            this.Players.TryGetValue(actorNr, out p);

            switch (eventCode)
            {
                case Player.EV_MOVE:
                    p.SetPosition((Hashtable)photonEvent[(byte)LiteEventKey.Data]);
                    break;
                //[...]
            }
        }

        //in Player.cs:
        internal void SetPosition(Hashtable evData)
        {
            this.posX = (byte)evData[(byte)STATUS_PLAYER_POS_X];
            this.posY = (byte)evData[(byte)STATUS_PLAYER_POS_Y];
        }
    
~~~~

Aside from the client, the server can also send any event as needed.
Check out the methods used in HandleRaiseEvent and use them in your
context.

### Properties

In Lite, Properties are Hashtables which can be set and fetched by
players. Properties are always attached to an actor or a room. They can
be used to store a room's setup, the players' names or anything else
that should be available for the clients.

You could use the Operations SetProperties and GetProperties and any
client might set any value. As shortcut, Join has an optional parameter
to set properties when you enter a room. That makes them immediately
available.

To avoid that each join re-sets the room's properties, Join will only
set them if the room is new. So, even if everyone sets room-properties
on join, only the first join sets them.

When you set properties, Lite gives you the option to broadcast them in
an event. This will be sent to the other users in a room, providing them
with the new property value.
