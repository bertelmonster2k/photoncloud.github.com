---
layout: article
title: Hello World - Part 2
categories: [photon-server, getting_started]
tags: [how-to, setup, installation]
---
{% include globals %}

Hello World - Part 2
====================

In Part 1 we introduced some basic concepts of the client api:
PhotonPeer, Service(), Connect() and the listener/callback design.
Buildling on the application of part 1 (initial connection setup) we
will look into how to use this connection to create a simple chat, where
the application joins a room and sends a "Hello World!" message to the
other connected clients.

We are going to look into two sets of concepts:

1.  -   **Operations** are remote procedure calls with a request and a
        response. An operation request consists of an **OperationCode**
        and **Parameters** (a Dictionary<byte, object\> containing the
        parameters). To send operation requests to the server
        **OpCustom()** of the peer instance is called. The operation
        result is returned to your application in the
        OnOperationResponse callback.
        Note: a) Sending the request and receiving the response are
        decoupled and fully asynchronous. b)the operation response is
        optional.
    -   **Events** are messages or notifications pushed to the client.
        When the peer receives an event the application is notified
        through the OnEvent callback

2.  -   **Rooms** group client connections or peers and facilitates the
        communication between peers. Peers can join a room and send
        events to all peers of the room or parts of them.
    -   **OpJoin** is the operation called by the client to join the
        room.
    -   **OpRaiseEvent** is the operation called to send events to other
        clients.

    Note: Rooms, OpJoin and OpRaiseEvent are implemented in the **Lite**
    Application. In part 3 we will have a brief look into the server
    side of these operations and see how we can customize the server
    based on Lite. Because the room concept is essential to many games
    this is the approach many developers take - although it's not the
    only one - Photon is committed to an **open architecture**
    philosophy. For more details on Applications see (tbd) and on Lite
    see (tbd).

Preparation: Refactoring the main flow
--------------------------------------

We will start a new C\#/Windows/Console project, name it Helloworld2,
add a reference to PhotonDotNet.dll and copy the Program.cs of part 1:

Next we'll refactor the code in main as follows

-   we will move the core flow into the "Program" instance method Run()
    (line 13).
-   Leaving a small stub of main just to create the instance and call
    Run() (line 3).
-   peer is declared as an instance variable and initialized in the
    constructor of "Program" (line 6).

~~~~ {.code}

        static void Main(string[] args)
        {
            new Program().Run();
        }

        PhotonPeer peer;

        public Program()
        {
            this.peer = new PhotonPeer(this);
        }

        void Run()
        {
            if (peer.Connect("localhost:5055", "Lite"))
            {
                do
                {
                    //Console.WriteLine(".");
                    peer.Service();
                    System.Threading.Thread.Sleep(500);
                }
                while (!Console.KeyAvailable);
            }
            else
                Console.WriteLine("Unknown hostname!");
            Console.WriteLine("Press any key to end program!");
            Console.ReadKey();
        }
~~~~

Note: in line 19 we commented the "." output. Optionally you can replace
it by Debug.Write("."); to make sure your loop is working. To see this
check your tab-output in you Visual Studio.

Operations and Events
---------------------

Now we are done with the preparation work we can start to see how we
send an operation request to the server.

The flow we are implementing once we initialized the connection:

1.  when status changes to connected: send the operation join request to
    join the room "MyRoomName". (in **OnStatusChanged**)
2.  when receiving the result "join ok": send the operation raise event
    request with the event code 101 and a "Hello World" message. (in
    **OnOperationResponse**)
    Note: usually you would define event codes centrally for your
    application in an enumeration for instance. Because we are using
    Lite features the codes 255-251 (Join, Leave, SetProperties,
    GetProperties) are predefined.
3.  when receiving an event with the code 101: print the received
    message to the console. (in **OnEvent**)

### OnStatusChanged

To start with the implementation of the flow we described above the
first change to the code we'll be to add a switch for the **statusCode**
we get notified with and a case block for the **StatusCode.Connect** to
send an **OperationRequest** to join a room with the name "MyRoomName"
(lines 6-11):

-   where we create a hastable **opParams** containing the parameters
    (lines 8+9).
-   and call **peer.OpCustom** with the **OperationCode** and passing in
    the **opParams** Dictionary<byte, object\> (line 10).

~~~~ {.code}

public void OnStatusChanged(StatusCode statusCode)
{
    Console.WriteLine("\n---OnStatusChanged:" + statusCode);
    switch (statusCode)
    {
        case StatusCode.Connect:
            Console.WriteLine("Calling OpJoin ...");
            Dictionary opParams = new Dictionary();
            opParams[(byte)LiteOpKey.GameId] = "MyRoomName";
            peer.OpCustom((byte)LiteOpCode.Join, opParams, true);
            break;
        default:
            break;
    }
}
~~~~

### OnOperationResponse

For an easy way to print out readable operation codeswe will define the
following enum: (this won't be necessary for 3.0):

~~~~ {.code}

    enum OpCodeEnum : byte
    {
        Join = 255,
        Leave = 254,
        RaiseEvent = 253,
        SetProperties = 252,
        GetProperties = 251
    }
~~~~

Next we change the OnOperationResponse callback as follows:

1.  We check the **operationResponse.ReturnCode** if it failed (not 0)
    we exit the method (lines 3,8).
2.  Add a switch-case for OperationCode
    (**operationResponse.OperationCode**) **LiteOpCode.Join** (lines
    11,13)
3.  We create a RaiseEvent operation request (LiteOpCode.RaiseEvent).
    With the parameters LiteOpKey.Code = 101 and LiteOpKey.Data = "Hello
    World" (our message) (lines 18-21).

~~~~ {.code}

public void OnOperationResponse(OperationResponse operationResponse)
{
    if (operationResponse.ReturnCode == 0)
        Console.WriteLine("\n---OnOperationResponse: OK - " + (OpCodeEnum)operationResponse.OperationCode + "(" + operationResponse.OperationCode + ")");
    else
    {
        Console.WriteLine("\n---OnOperationResponse: NOK - " + (OpCodeEnum)operationResponse.OperationCode + "(" + operationResponse.OperationCode + ")\n ->ReturnCode=" + operationResponse.ReturnCode + " DebugMessage=" + operationResponse.DebugMessage);
        return;
    }

    switch (operationResponse.OperationCode)
    {
        case (byte)LiteOpCode.Join:
            int myActorNr = (int)operationResponse.Parameters[LiteOpKey.ActorNr];
            Console.WriteLine(" ->My PlayerNr (or ActorNr) is:" + myActorNr);

            Console.WriteLine("Calling OpRaiseEvent ...");
            Dictionary opParams = new Dictionary();
            opParams[LiteOpKey.Code] = (byte)101;
            opParams[LiteOpKey.Data] = "Hello World!";
            peer.OpCustom((byte)LiteOpCode.RaiseEvent, opParams, true);
            break;
    }
}
~~~~

When you run this code the ouput will look like this:

C:\\...\\HelloWorld2\\bin\\Debug\>HelloWorld2.exe\
 \
 ---OnStatusChanged:Connect\
 Calling OpJoin ...\
 \
 ---OnOperationResponse: OK - Join(90)\
 -\>My PlayerNr (or ActorNr) is:1\
 Calling OpRaiseEvent ...\
 \
 ---OnOperationResponse: NOK - RaiseEvent(253)\
 -\>ReturnCode=-1 DebugMessage=Wrong parameter type 245
(RaiseEventRequest.Data): should be Hashtable but received String\

As you can see the first operation (Join) returned **OK**. It also
returned the actor number 1 (**-\>My PlayerNr (or ActorNr) is:1**). If
you start a second Hellowrold2.exe you will see a "2" instead. The next
client would get a "3".

Note: starting and stopping clients might lead to higher actor nr
values. The server recognizes the disconnect after a certain timeout and
removes those peers. Usually a client would send a disconnect on
shutdown - which is the recommended way so that others "see" faster a
peer disconnected - a peer removed from the room triggers a leave event
broadcasting it to all peers connected.

The second operation OpRaiseEvent failed. This is because the parameter
**Data** is expected to be a hashtable. So we will change our code as
follows (lines 10-13):

~~~~ {.code}

    switch (operationResponse.OperationCode)
    {
        case LiteOpCode.Join:
            int myActorNr = (int)operationResponse.Parameters[LiteOpKey.ActorNr];
            Console.WriteLine(" ->My PlayerNr (or ActorNr) is:" + myActorNr);

            Console.WriteLine("Calling OpRaiseEvent ...");
            Dictionary opParams = new Dictionary();
            opParams[LiteOpKey.Code] = (byte)101;
            //opParams[LiteOpKey.Data] = "Hello World!"; //<- returns an error, server expects a hashtable
            Hashtable evData = new Hashtable();
            evData[(byte)1] = "Hello Wolrd!";
            opParams[LiteOpKey.Data] = evData;
            peer.OpCustom((byte)LiteOpCode.RaiseEvent, opParams, true);
            break;
    }
~~~~

Running the code with the changes we just made you will notice the
OnOperationResponse for RaiseEvent doesn't appear anymore - this is
because the server only sends a response in case of an error!

Note: the key (byte)1 of evData has nothing to do with the parameter
opParams[LiteOpKey.Code] beeing (byte)101(line 9).

Hint: Because events tend to be sent very often it's recommended to use
either byte or short keys in the evData hashtable.

### OnEvent

For an easy way to print out readable operation codeswe will define the
following enum: (this won't be necessary for 3.0):

~~~~ {.code}

    enum EvCodeEnum : byte
    {
        Join = 255,
        Leave = 254,
        PropertiesChanged = 253
    }   
~~~~

The last change we're making is to display the message we receive in the
eventData when we receive the event with code = 101:

~~~~ {.code}

public void OnEvent(EventData eventData)
{
    Console.WriteLine("\n---OnEvent: " + (EvCodeEnum)eventData.Code + "(" + eventData.Code + ")");

    switch (eventData.Code)
    {
        case 101:
            int sourceActorNr = (int)eventData.Parameters[LiteEventKey.ActorNr];
            Hashtable evData = (Hashtable)eventData.Parameters[LiteEventKey.Data];
            Console.WriteLine(" ->Player" + sourceActorNr + " say's: " + evData[(byte)1]);
            break;
    }
}
~~~~

If you launch two clients now you'll see the following

**the first client:**

C:\\...\\HelloWorld2\\bin\\Debug\>HelloWorld2.exe\
 \
 ---OnStatusChanged:Connect\
 Calling OpJoin ...\
 \
 ---OnOperationResponse: OK - Join(255)\
 -\>My PlayerNr (or ActorNr) is:1\
 Calling OpRaiseEvent ...\
 \
 ---OnEvent: Join(255)\
 \
 ---OnEvent: Join(255)\
 \
 ---OnEvent: 101(101)\
 -\>Player2 say's: Hello Wolrd!\

**the second client:**

C:\\...\\HelloWorld2\\bin\\Debug\>HelloWorld2.exe\
 \
 ---OnStatusChanged:Connect\
 Calling OpJoin ...\
 \
 ---OnOperationResponse: OK - Join(255)\
 -\>My PlayerNr (or ActorNr) is:2\
 Calling OpRaiseEvent ...\
 \
 ---OnEvent: Join(255)\
 \

The first client receives 3 events:

1.  the join event triggered by it's own join.
2.  the join event triggered by the joining of the second client.
3.  the 101 event the second client sent to **all others** in the room

**all others** is the default behavior of OpRaisEvent (for more details
see sdk documentation of LitePeer).

The second client only receives one join event (triggered by it's own
join).

In this tutorial we only used **PhotonPeer**. The client sdk also
includes a **LitePeer** to ease the use of Lite features. It extends the
PhotonPeer so that to join a room with LitePeer you only need to call
peer.OpJoin("MyRoomName"). To see how this works just replace PhotonPeer
by LitePeer. An other powerfull Lite features worth looking into are
**Properties**. In Lite you can set room and player (or actor)
properties. As an example you could set initial configuration values
(e.g. te game map, difficulty, ...) as room properties or the players
nicknames as actor properties. (for more details see the client sdk
documentation and Lite tbd).

Final Demo Code
---------------

In the code that follows we added a couple of lines to increase
debug-output that should help you dig deeper into the details of how
Photon and Lite work.

~~~~ {.code}

using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

using ExitGames.Client.Photon;
using ExitGames.Client.Photon.Lite;
using System.Diagnostics;
using System.Collections;

namespace HelloWorld2
{
    class Program : IPhotonPeerListener
    {
        static void Main(string[] args)
        {
            new Program().Run();
        }

        PhotonPeer peer;

        public Program()
        {
            this.peer = new PhotonPeer(this);
        }

        void Run()
        {
            //DebugLevel should usally be ERROR or Warning - ALL lets you "see" more details of what the sdk is doing.
            //Output is passed to you in the DebugReturn callback
            peer.DebugOut = DebugLevel.ALL; 
            if (peer.Connect("localhost:5055", "Lite"))
            {
                do
                {
                    Debug.Write("."); //allows you to "see" the game loop is working, check your output-tab when running from within VS
                    peer.Service();
                    System.Threading.Thread.Sleep(50);
                }
                while (!Console.KeyAvailable);
            }
            else
                Console.WriteLine("Unknown hostname!");
            Console.WriteLine("Press any key to end program!");
            Console.ReadKey();
            //peer.Disconnect(); //<- uncomment this line to see a faster disconnect/leave on the other clients.
        }

        #region IPhotonPeerListener Members
        public void DebugReturn(DebugLevel level, string message)
        {
            // level of detail depends on the setting of peer.DebugOut
            Debug.WriteLine("\nDebugReturn:" + message); //check your output-tab when running from within VS
        }

        public void OnOperationResponse(OperationResponse operationResponse)
        {
            if (operationResponse.ReturnCode == 0)
                Console.WriteLine("\n---OnOperationResponse: OK - " + (OpCodeEnum)operationResponse.OperationCode + "(" + operationResponse.OperationCode + ")");
            else
            {
                Console.WriteLine("\n---OnOperationResponse: NOK - " + (OpCodeEnum)operationResponse.OperationCode + "(" + operationResponse.OperationCode + ")\n ->ReturnCode=" + operationResponse.ReturnCode
                  + " DebugMessage=" + operationResponse.DebugMessage);
                return;
            }

            switch (operationResponse.OperationCode)
            {
                case LiteOpCode.Join:
                    int myActorNr = (int)operationResponse.Parameters[LiteOpKey.ActorNr];
                    Console.WriteLine(" ->My PlayerNr (or ActorNr) is:" + myActorNr);

                    Console.WriteLine("Calling OpRaiseEvent ...");
                    Dictionary opParams = new Dictionary();
                    opParams[LiteOpKey.Code] = (byte)101;
                    //opParams[LiteOpKey.Data] = "Hello World!"; //<- returns an error, server expects a hashtable

                    Hashtable evData = new Hashtable();
                    evData[(byte)1] = "Hello Wolrd!";
                    opParams[LiteOpKey.Data] = evData;
                    peer.OpCustom((byte)LiteOpCode.RaiseEvent, opParams, true);
                    break;
            }
        }

        public void OnStatusChanged(StatusCode statusCode)
        {
            Console.WriteLine("\n---OnStatusChanged:" + statusCode);
            switch (statusCode)
            {
                case StatusCode.Connect:
                    Console.WriteLine("Calling OpJoin ...");
                    Dictionary opParams = new Dictionary();
                    opParams[LiteOpKey.GameId] = "MyRoomName";
                    peer.OpCustom((byte)LiteOpCode.Join, opParams, true);
                    break;
                default:
                    break;
            }
        }

        public void OnEvent(EventData eventData)
        {
            Console.WriteLine("\n---OnEvent: " + (EvCodeEnum)eventData.Code + "(" + eventData.Code + ")");

            switch (eventData.Code)
            {
                case LiteEventCode.Join:
                    int actorNrJoined = (int)eventData.Parameters[LiteEventKey.ActorNr];
                    Console.WriteLine(" ->Player" + actorNrJoined + " joined!");

                    int[] actorList = (int[])eventData.Parameters[LiteEventKey.ActorList];
                    Console.Write(" ->Total num players in room:" + actorList.Length + ", Actornr List: ");
                    foreach (int actorNr in actorList)
                    {
                        Console.Write(actorNr + ",");
                    }
                    Console.WriteLine("");
                    break;

                case 101:
                    int sourceActorNr = (int)eventData.Parameters[LiteEventKey.ActorNr];
                    Hashtable evData = (Hashtable)eventData.Parameters[LiteEventKey.Data];
                    Console.WriteLine(" ->Player" + sourceActorNr + " say's: " + evData[(byte)1]);
                    break;
            }
        }

        #endregion
    }
}
~~~~
