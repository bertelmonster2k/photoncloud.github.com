---
layout: article
title: Load Balancing Demo Tutorial
categories: [photon-cloud, getting_started]
tags: [sample, quickstart, how-to]
---
{% include globals %}

Photon is a light-weight and sophisticated way to use middleware to free
you from all the hassle of programming network code. Common operations
and methods already implemented within the Photon middleware will give
you the time to focus on gameplay logic and the more interesting aspects
of turning your idea into a success. See for yourself!

In this post we'll show you some chosen excerpts from our Photon Cloud
"Loadbalancing Demo". The demo that comes with our SDK shows you how to
implement some of the typical use cases you will face when trying to add
multiplayer capabilities to your application. In the following we will
take a look at the basic Photon specific operations and how they are
used in the context of this demo. Easy readable C\# code samples will be
used in order to illustrate what Photon enables you to do right out of
the box.

## Connect To Master Server

The first thing you need to do is to connect your client to the cloud.
We call this process connecting to the Master Server. From there on the
Master Server will take care of all the clients' transfers to game
servers, load-balancing within the cloud and the available rooms.

Let's delve right into the code of our demo:

~~~~ {.code}
                        
                    public DemoClient(): base()
                    {
                        this.PlayerName = "Player" + this.randomNumber.Next(1, 100);
                        this.ConnectToMaster("yourAppIdHere", 
                                                      "v0.1", 
                                                  PlayerName); 
                    }
                        
                    
~~~~

Let's take a closer look at the arguments used in the "ConnectToMaster"
operation:

1.  **string ApplicationID** It identifies your application within our cloud system. 
    If you don't already have an Application ID you
    can get it [here](https://www.exitgames.com/Download/Photon) for free.
    You need to insert a valid Application ID in order for this demo to
    work.
2.  **string VersionNumber** The second parameter is the version number. The version number
    is a string that you can choose by yourself - it is used to make sure
    that only clients with the same version number get matched and can
    communicate with each other. This allows you to have different builds of
    your application online and playable!
3.  **string PlayerName** The third parameter is the name of the player, we won't get
    into the rocket science involved in this complex construct. Suffice to
    say it is used by other players to identify the person playing.

After you are connected, the Photon Cloud is ready to do your bidding.
You don't have to micromanage from here on, everything is now handled
for you! In the diagram below you can see that everything in the grey
square is organized by Photon Cloud. The client only needs to send
simple operations, like the ones next to the arrows.

<p>
<figure>
<img src="{{ IMG }}/JoinMSCloud.png" />
<figcaption>The game servers that host the rooms reside in the cloud.</figcaption>
</figure>
</p>

Now that we are connected to the Master Server we can list, create and
join rooms. At this point players can't communicate or interact with
each other. This is where the concept of rooms comes into play. Read on
to see what you need to do to connect the players to each other.

## Create Room / Join Room

A room can be considered as a seperate area where players "meet". If
they are in the same room, players can send and receive events from
other players, change/update properties of the room etc. Creating and
joining rooms can be done with only a few lines of code.

~~~~ {.code}
                        
                    public void CreateNewRoom(string name)
                    {
                    Hashtable customGameProperties = new Hashtable() 
                    { { "map", "blue" }, { "maxNumberPlayers", 16 } };
                    this.OpCreateRoom(name, true, true, 2, 
                                     customGameProperties, 
                        this.LocalPlayer.CustomProperties);
                    }
                        
                    
~~~~

In line 10/11 we create a set of customGameProperties. These can be
changed anytime and they are inherent to this specific room. Think of it
as a group of attributes you can freely define to use them inside your
game logic. For example you could set up a game with a certain map
activated or restrict access for players with lower level. The
possibilites are vast and you can even use it to build your [own
matchmaking system](https://www.exitgames.com/Download/Photon)!

In the following lines we use the "OpCreateRoom" operation to create and
open up the room. In order the arguments are as follows:

1.  **string roomName** The name of the room, used for identifying and
    joining the room.
2.  **bool isVisible** This variable determines if the room is in the
    list of visible rooms that can be seen from the lobby (i.e. players
    that are connected to the Master Server, but do not reside in a
    room). Important is that these rooms can still be joined, as long as
    the client knows the exact name of the room.
3.  **bool isOpen** Determines if the room can be joined by a client.
    Clients already in the room are unaffected when this variable is
    changing. However, they can't rejoin the room after leaving it, as
    long as isOpen is false.
4.  **byte maxPlayers** Determines the maximum amount of players in this
    room. If set to 0 the number is unlimited. Note however that if your
    planning to have a large amount of users **in one room** you should
    take a look at our [MMO
    Application](https://www.exitgames.com/Download/Photon)!
5.  **Hashtable customGameProperties** As already discussed above, this
    is a set of customProperties which you can use for various means.
6.  **Hashtable customPlayerProperties** Analog to the properties of a
    room, these are properties of the players. These settings are not
    bound to the room, but instead to the client itself. So when
    traversing rooms, your customPlayerProperties will stay with the
    client/player for the time being.

<figure>
<img src="{{ IMG }}/PropertiesCloud.png" />
<figcaption>Example for properties.</figcaption>
</figure>

Now that we successfully created a room, it's time for other clients to
join! Joining a room is quick and easy and shouldn't need any further
explanations as the operation is speaking for itself:

~~~~ {.code}
                        
                    public void JoinSelectedRoom(string name)
                    {
                    this.OpJoinRoom(name, this.LocalPlayer.CustomProperties);
                    }                    
~~~~

Let's see how the players can now interact with each other.

## Send Events

For clients to interact with each other, we use a simple event system.
Using events is the main way of sending and receiving fast and reliable
information from and to players inside a given room. All the necessary
data for your game logic can be send between clients this way. You can
even customize the way of exchanging information by specifying a
protocol (UDP vs. TCP or reliable UDP vs. unreliable UDP) with simple
parameters, depending on your needs.

<figure>
<img src="{{ IMG }}/SendEventCloud.png" />
<figcaption>Sending Events</figcaption>
</figure> 
Events will be distributed among participants in the room. You can decide if
you want to sent events to a specific list of players, groups or
everyone.

Let's take a look at an event we use in the Windows Phone version of the
load balancing demo:

~~~~ {.code}
                        
                    public void SendSomeEvent()
                    {
                        Hashtable eventContent = new Hashtable();
                        eventContent[(byte)10] = "my data";                 
    
                        this.OpRaiseEvent(1, eventContent, false, 0);
                    }
                        
                    
~~~~

The OpRaiseEvent takes the following arguments:

1.  **byte eventCode** The event code specifies the actual event that you
    want to raise. There are predefined events by Photon starting from 255
    and then moving downwards. Beginning with 1 you can define your own
    events to use in your game logic. For a complete list of predefined
    events check [here](https://www.exitgames.com/Download/Photon)!
2.  **Hashtable evData** You can fill this hashtable with all the data you
    need to transfer. This is the central data-structure you will use to
    exchange information between clients. However, recurring and
    standardized steps for your game logic (e.g. end of turn) should rather
    be send as events.
3.  **bool sendReliable** When you set this flag to "true", you switch from
    UDP to reliable UDP. This means that any packages lost during
    transmission will be re-send and that the clients will make sure that
    the packages will be interpreted in the order they've been sent. Not
    that this might affect performance in a negative way as this extra steps
    will add additional messages and data burden to overall communication.
4.  **byte channelId** You can use different channels to group and prioritize events you are
    sending. We'll illustrate this functionality in a small example: Let's
    say you are using channel 1 to send relevant information about the
    position of players. You enabled reliable UDP in your game, because you
    need the extra reliability in the type of game your are envisioning. At
    some point channel 1 gets crowded by many messages because some players
    got delay and have a bad connection with a noticeable amount of packet
    loss, so a lot of messages have to be re-send. If you would now enqueue
    another important event (like the end of round), it might take some time
    for it to be acknowledged since the clients are still busy receiving all
    the position updates from channel 1. So if you dispatch an event now
    with channelId 0 it will take precedence over all the other queued
    messages. By making smart use of this feature you can raise the
    perceived responsiveness and ensure a correct flow of your game logic,
    even under mediocre connection conditions.

Last but not least there are additional overloads that you can use to
specify the receiver groups. Please take a look
[here](https://www.exitgames.com/Download/Photon) to find out more about
that feature. Thanks for taking your time reading this post! If you have
any questions or comments about this article please come over to our
forum and tell us, we really appreciate your feedback!
