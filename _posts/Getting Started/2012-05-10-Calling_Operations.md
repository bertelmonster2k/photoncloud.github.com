---
layout: article
title: Calling Operations
categories: [photon-server, photon-cloud, getting_started]
tags: [overview, operations]
---
{% include globals %}

Operations in the LitePeer class
--------------------------------

As described in the "Basics" section, Operations are Remote Procedure
Calls, defined by some Photon Application.

Because we use the Lite Application most often for demos and to make
your life easier, the client APIs include a LitePeer class (or similar
construct), which provides methods to call operations on the server.

For example, LitePeer.OpJoin() wraps up the call of Operation Join. The
code below will call OpJoin when the connection is established:

~~~~ {.code}
    
        public void PeerStatusCallback(StatusCode returnCode)
        {
            // handle returnCodes for connect, disconnect and errors (non-operations)
            switch (returnCode)
            {
                case StatusCode.Connect:
                    this.DebugReturn("Connect(ed)");
                    this.peer.OpJoin(this.gameID);
                    break;
                //[...]
    
~~~~

The LitePeer covers all Lite Application Operations this way.

### Operation Results from Photon

Per Operation, the application on the server can send a result. Imagine
a "GetHighscores" Operation and the use for a result becomes obvious and
would contain a list of scores. Other RPCs, like RaiseEvent, can safely
omit the result if it's not useful.

Getting the result of an Operation takes a moment in general, as there
is network lag. Because of that, any result is provided asynchronously
by a call to IPhotonPeerListener.OperationResult().

Per platform, the result callback might look different. The following
code is from the C\# callback interface, which a game must implement:

~~~~ {.code}
    
    public interface IPhotonPeerListener
    {
        //[...]
        void OperationResult(byte opCode, int returnCode, Hashtable returnValues, short invocID);
    
~~~~

The callback OperationResult() is only called when your game calls
peer.DispatchIncomingCommands(). This makes it easier to get the
callback in the right thread context. In most cases it is enough to call
DispatchIncomingCommands() every few frames in your game loop.

### Custom Operations

Custom Operations are just Operations which are not defined by Exit
Games in Lite, LiteLobby or the MMO Application. Basically, any
Operation that's not covered by the LitePeer API is "custom".

If you look behind the scenes you will see that even non-custom
Operations, like Join, use the very same methods! As example, here is
the code from LitePeer Join:

~~~~ {.code}
    
        public virtual short OpJoin(string gameName, Hashtable gameProperties, Hashtable actorProperties, bool broadcastActorProperties)
        {
            Hashtable params = new Hashtable();
            params[(byte)LiteOpKey.Asid] = gameName;
            if (actorProperties != null)
            {
                params[(byte)LiteOpKey.ActorProperties] = actorProperties;
            }

            if (gameProperties != null)
            {
                params[(byte)LiteOpKey.GameProperties] = gameProperties;
            }

            if (broadcastActorProperties)
            {
                params[(byte)LiteOpKey.Broadcast] = broadcastActorProperties;
            }

            return OpCustom((byte)LiteOpCode.Join, params, true, 0, false);
        }
    
~~~~

As you see, OpJoin internally uses OpCustom to be sent. The method is
just a wrapper to streamline the usage. This is what you should do as
well.

The call to OpCustom shows the parts needed to send an Operation:

-   **OperationCode**: a single byte to identify an Operation (to
    minimize overhead). (byte)LiteOpCode.Join equals 90, as example.
-   **Parameters**: is the set of parameters the Operation Object
    expects on the server side. Again, bytes are used to minimize
    overhead per parameter.
-   **SendReliable**: chooses if an operation must arrive at the server
    or might become lost. Data that's replaced quickly can be
    unreliable. Join, Authentication or similar are better sent
    reliable.
-   **ChannelId**: is the sequence this Operation is placed into for
    ordering.
-   **Encrypt**: optionally encrypts data between the client and the
    server. Should be used carefully, as it takes away some performance
    and it not always needed.

The latter three values are not exactly Operation parts, obviously.
SendReliable, ChannelId and Encrypt are communication settings which can
be changed on a per-operation basis.

The byte codes for the Operation, parameters and result values are
defined on the server side. To call it, you need to use those on the
client correspondingly.

### OpCustom() Return Value

In the code examples above, we didn't use the return value of OpCustom
(or OpJoin) yet, but you know by now it's not the result of an
Operation.

When you call OpCustom the immediate result will either note an error,
or supply you with an "Invocation ID":

-   If an operation can't be sent (e.g. because the game is not
    connected), the returned value will be: -1. If this happens, you
    should check the debug hints that the client library provides by
    calling DebugReturn().
-   If the returned value is not -1, it's an "Invocation ID", which
    identifies "this" request call. The response to it will have a
    matching Invocation ID, so you know which request and response
    belong together. If you don't need to match calls and results, you
    can ignore the returned Invocation ID.

### Defining New Operations

In most cases, the definition of new Operations is done server-side, so
the process to implement and use those is explained in that context.
Read: "[Adding Operations](/adding_operations)".
