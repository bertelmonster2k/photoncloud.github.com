---
layout: article
title: Lite Lobby Chat Demo
categories: [photon-server, tutorials]
tags: [lite, lobby, sample, how-to]
---
{% include globals %}

Chatroom and Lobby Demo
-----------------------

This demo shows some of the things you can do with Photon's "Lite Lobby"
application, purely from the client side. No server programming is
involved here. The client is written in C\# and uses the Unity Engine
(which also has a free edition) as framework and GUI.

The Lite Lobby Chatroom Demo part of the Unity Client SDK v6.2.0 (and
up). [Get the SDK and demo here](http://www.exitgames.com/download/).

#### Quick Walkthrough

First off, let's go through the screens. When you start, the demo will
automatically connect to Photon on the local machine. If this fails (and
whenever connection is lost), it will display an error message.

You will be asked to enter a name and "Proceed to Lobby". A short text
explains "debug mode" keys to control multiple clients. This will come
in handy in a minute. Enter the lobby. As noone else is on your server,
you should create a room. Enter a name and click "Enter". The next
screen is a simple chat: Messages are entered at the bottom, users
listed on the right side and there is a button to leave. Now, more users
would be nice.

![](../img/Demo-LiteLobbyChat-EmptyChat.jpg) Almost empty chat

Of course, you could start the demo several times to get more people in
the chat. To make things easier, you can instantiate a chat client by
pressing Insert and switch between the simulated clients with Page Up
and Page Down key.

Press: Insert, then PageDown. You will see the name input screen once
more. Keep in mind: now the input controls another client. Chose another
name and press the button and now you will see a button. The lobby will
create a button for each room that's in use. You can join that room or
create another.

#### Development Toolkit

In this demo (and in general) things get easier to develop if you don't
have to run a game many times to test multiplayer. For that reason, we
will create clients that maintain their connection and state but not
their input. Input is something we do in a central place and apply to an
active client.

The PhotonClient is the lowest level we can automate a client. This can
be re-used in other applications and will simply keep a state and
connection. Extending it is the ChatPhotonClient, which adds
chat-related states, a few methods and most of all, it handles our
custom chat events.

To make this run in Unity, we need to attach the ChatGui script and an
initial ChatPhotonClient to any game object. We used the empty object
"PhotonScripts". The public values of the initial ChatPhotonClient let
us edit the server before we start the demo. While running, we can check
each client's state, name, room and control if we want it's debug out in
the console. This setup makes things transparent and we might even save
a few debugging sessions.

![](../img/Demo-LiteLobbyChat-ScriptsInEditor.jpg) Script values at
runtime

Missing in the client library is a special peer class for LiteLobby,
which is more or less treated like any game you would develop. To fill
the gap, we implement a LiteLobbyPeer class, which is used instead of
the LitePeer. Once you create your own operations for your game, you
should also create a fitting "myGamePeer" class.

#### The Lobby

The Lite Lobby application defines two types of rooms: games and
lobbies. But there is only one operation to join a room and there is no
parameter to chose. The secret is, that Lite Lobby creates a lobby when
the roomname to join ends on "\_lobby" and a game room in any other
case. An optional parameter can be used to tell a game room to update a
certain lobby. Read about the server side on page: [Lite Lobby
Concepts](/liteandlitelobbyaddon/litelobbyconcepts).

The additional features of Lite Lobby are currently not implemented by
the client library. This where our LiteLobbyPeer class comes to use. It
defined the needed operation- and event-codes and the codes for
parameters and event-content already.

As all rooms are created on the fly, it's up to the client to create
lobbies, too. This demo uses a single lobby called "chat\_lobby" but you
could also add a selection-screen for a list of lobbies. As Lite Lobby
does not support listing of lobbies out of the box, you could either add
a hard-coded list to select or change the server code to support it.

#### The Chatroom

We already noticed that the operation Join creates lobbies and games
alike. When we create a room, we might set a lobby to report to.
LiteLobbyPeer implements this version of operation Join as
OpJoinFromLobby. It wraps up the required parameters and even allows us
to set actor properties (but not room properties, which we don't use in
this demo anyways).

~~~~ {.code}
    
    public virtual short OpJoinFromLobby(string gameName, string lobbyName, Hashtable actorProperties, bool broadcastActorProperties)
    {
        if (this.DebugOut >= DebugLevel.ALL)
        {
            this.Listener.DebugReturn(DebugLevel.ALL, String.Format("OpJoin({0}/{1})", gameName, lobbyName));
        }

        // All operations get their parameters as key-value set (a Hashtable)
        Hashtable opParameters = new Hashtable();
        opParameters[(byte)LiteLobbyOpKey.RoomName] = gameName;
        opParameters[(byte)LiteLobbyOpKey.LobbyName] = lobbyName;

        if (actorProperties != null)
        {
            opParameters[(byte)LiteOpKey.ActorProperties] = actorProperties;
            if (broadcastActorProperties)
            {
                opParameters[(byte)LiteOpKey.Broadcast] = broadcastActorProperties;
            }
        }

        return OpCustom((byte)LiteLobbyOpCode.Join, opParameters, true);
    }
    
~~~~

Inside the chatroom, we want to know who is there and we want to send
messages to everyone in the room.

#### Using Properties for Usernames

In the Lite Lobby application, properties are key-value sets you can use
to describe rooms and actors. This demo uses actor properties to store
usernames. Each client sets it's name in the Join operation and uses
broadcast to sent the new name to the others.

~~~~ {.code}
    
    public void JoinRoomFromLobby(string roomName)
    {
        this.RoomName = roomName;
        this.ChatState = ChatStateOption.JoiningChatRoom;
        this.ActorProperties = new Hashtable();

        Hashtable props = new Hashtable() { { ChatActorProperties.Name, this.UserName } };
        this.Peer.OpJoinFromLobby(this.RoomName, this.LobbyName, props, true);
    }
    
~~~~

A new user won't get a join event for players who were in the room
before, so we use GetProperties to get everyones name. It's result is
the complete property set at the time being. From now on, we just add
properties when someone joins or remove them on leave.

~~~~ {.code}
    
    switch (eventCode)
        {
            // this client or any other joined the room (lobbies will not send this event but chat rooms do)
            case (byte)LiteEventCode.Join:
                this.DebugReturn(SupportClass.HashtableToString(photonEvent));

                // update the list of actor numbers in room
                this.ActorNumbersInRoom = (int[])photonEvent[(byte)LiteOpKey.ActorList];

                // update the list of actorProperties if any were set on join
                Hashtable actorProps = photonEvent[(byte)LiteOpKey.ActorProperties] as Hashtable;
                if (actorProps != null)
                {
                    this.ActorProperties[originatingActorNr] = actorProps;
                }
                break;

            // some other user left this room - remove his data
            case (byte)LiteEventCode.Leave:
                // update the list of actor numbers in room
                this.ActorNumbersInRoom = (int[])photonEvent[(byte)LiteOpKey.ActorList];

                // update the list of actorProperties we cache
                if (this.ActorProperties.ContainsKey(originatingActorNr))
                {
                    this.ActorProperties.Remove(originatingActorNr);
                }
                break;
    
~~~~

#### Sending and Receiving Chat Messages

The Lite Lobby application gives us rooms and events and a way to raise
them. In our simple chat demo, we don't need the server to understand
what the clients are talking about.

Custom events are defined by the client and handled by the clients. The
server just passes them on when a client calls RaiseEvent. In our case,
we select a free event code and add in our text line.

~~~~ {.code}
    
    public void SendChatMessage(string line)
    {
        Hashtable chatEvent = new Hashtable();  // the custom event's data. content we want to send
        chatEvent.Add((byte)ChatEventKey.TextLine, line);   // add some content
        this.Peer.OpRaiseEvent((byte)ChatEventCode.Message, chatEvent, true);   // call raiseEvent with our content and a event code

        // because Photon won't send this event back to this client, we will add the chat line locally
        this.ChatLines.AppendLine(String.Format("{0}: {1}", this.UserName, line));
    }
    
~~~~

Lite Lobby will create an event from our data and adds the origin to it.
Other users in a room will receive the event and know who sent it. To
keep things lean, this is done by the actorNumbers.

Our sent text line is the data of our custom event. We fetch the name
and put the line to the chat buffer.
