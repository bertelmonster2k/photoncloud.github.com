---
layout: article
title: Hello World - Part 1
categories: [photon-server, getting_started]
tags: [how-to, setup, installation]
---
{% include globals %}

Hello World - Part 1
====================

The purpose of this tutorial is to give you an introduction to the
client api and some of the basic concepts behind Photon. The tutorial is
divided in parts. We begin with a very simple demo which will be
refactored to more usable code step by step while digging deeper into
the different concepts of Photon.

Overview
--------

The application we will create is a simple windows console application
that connects to a locally hosted photon server.

Note: We assume you have followed the steps in "Photon in 5 Minutes" and
have a locally running Photon Server.

The basic concepts we will build the application on are:

-   To communicate with Photon Server we need a **PhotonPeer** where you
    call **Connect()** to initiate the communication.
-   Outgoing/Incoming messages are queued in the PhotonPeer. In order to
    transfer the messages over the wire and dispatch (incoming) them to
    your application **Service()** has to be called.
-   Changes in the status of the connection will be notified to the
    **IPhotonListener.OnStatusChanged**.

Hands-On
--------

### Project Setup

We will start with a C\#/Windows/Console project and name it
Helloworld1:

![](../img/HelloWorld-NewProject.png) New Project

We need to add a reference to PhotonDotNet.dll:

![](../img/HelloWorld-AddReference.png) Add PhotonDotNet.dll reference

Select the folder where you extracted the Photon-Dotnet SDK (tbd link to
downloads) and browse to lib/debug where you will find PhotonDotNet.dll

![](../img/HelloWorld-BrowseReference.png) Select Browse ...

The next step ist to add "using ExitGames.Client.Photon;"

~~~~ {.code}

using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

using ExitGames.Client.Photon;
~~~~

### OnStatusChanged

Now we will implement the listener where we receive the PhotonPeer
notifications. So the next step is adding the interface to the **Class
Program**:

![](../img/HelloWorld-AddInterface.png) Add listener interface

For now we will only look into **OnStatusChanged** where status changes
of the PhotonPeer are notified and ignore the other three callbacks. So
next we will replace the NotImplementedException as highlighted below:

~~~~ {.code}

        #region IPhotonPeerListener Members

        public void DebugReturn(DebugLevel level, string message)
        {
            //throw new NotImplementedException();
        }

        public void OnEvent(EventData eventData)
        {
            //throw new NotImplementedException();
        }

        public void OnOperationResponse(OperationResponse operationResponse)
        {
            //throw new NotImplementedException();
        }

        public void OnStatusChanged(StatusCode statusCode)
        {
            Console.WriteLine("OnStatusChanged:" + statusCode);
        }

        #endregion
    }
~~~~

### Peer Connect

So now we are getting to the main part of our tutorial. We add line 3 to
create an instance of our listener. Line 4 will create an instance of
the PhotonPeer and line 5 initiates the server connect to localhost on
port 5055. "Lite" is the default application on the server - we will
explain applications more in detail in part 3 (tbd link).

~~~~ {.code}

        static void Main(string[] args)
        {
            var listener = new Program();
            var peer = new PhotonPeer(listener);
            peer.Connect("localhost:5055", "Lite");

~~~~

### Calling Service

When running the code above - nothing would happen. We have to call
peer.Service() to trigger the network transfere and dispatching of
notifications:

~~~~ {.code}

            do
            {
                Console.WriteLine(".");
                peer.Service();
                System.Threading.Thread.Sleep(500);
            }
            while (true);
~~~~

Ok. We are done. Hit F5. If Photon Server is running you will see the
following:

![](../img/HelloWorld-Output1.png) Console output

That's it you are connected to the Photon Server in part 2 we will look
on how to start using the connection to the Photon Server.

Photon Server not found
-----------------------

If Photon Server is not running you will get a disconnect notification:

![](../img/HelloWorld-Output2.png) Console output

In the screenshot above you see the behavior when trying to connect
against a locally hosted Photon Server

OnStatusChanged:InternalReceiveException: Because the client is trying
to connect a not existent service on the same machine the windows socket
lib raises an exception.

The normal behavior connecting to a remote server where photon is not
running would a disconnect triggered after a timeout:

![](../img/HelloWorld-Output3.png) Console output

You can try this out simply by connecting to a site you now is not
running photon "google.com" for instance

Connect() returns false if the hostname is unknown. The state of the
peer doesn't change to connecting.

To see the host unknown behavior change your code as follows:

~~~~ {.code}

//          peer.Connect("localhost:5055", "Lite")
            if (peer.Connect("xxx:5055", "Lite"))
            {
                do
                {
                    Console.WriteLine(".");
                    peer.Service();
                    System.Threading.Thread.Sleep(500);
                }
                while (!Console.KeyAvailable);
            }
            else
                Console.WriteLine("Unknown hostname!");
~~~~

![](../img/HelloWorld-Output4.png) Console output

Final Demo Code
---------------

Hoover over the code section below an click on copy to clipboard
![](../img/HelloWorld-Copyicon.png) and replace the contents of your
Program.cs in your project.

~~~~ {.code}

using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

using ExitGames.Client.Photon;

namespace HelloWorld1
{
    class Program : IPhotonPeerListener
    {
        static void Main(string[] args)
        {
            var listener = new Program();
            var peer = new PhotonPeer(listener);

            if (peer.Connect("localhost:5055", "Lite"))
            //// if (peer.Connect("google.com:5055", "Lite"))
            //// if (peer.Connect("xxx:5055", "Lite"))
            {
                do
                {
                    Console.WriteLine(".");
                    peer.Service();
                    System.Threading.Thread.Sleep(500);
                }
                while (!Console.KeyAvailable);
            }
            else
                Console.WriteLine("Unknown hostname!");
            Console.ReadKey();
        }

        #region IPhotonPeerListener Members
        public void DebugReturn(DebugLevel level, string message)
        {
            //throw new NotImplementedException();
        }

        public void OnEvent(EventData eventData)
        {
            //throw new NotImplementedException();
        }

        public void OnOperationResponse(OperationResponse operationResponse)
        {
            //throw new NotImplementedException();
        }

        public void OnStatusChanged(StatusCode statusCode)
        {
            Console.WriteLine("OnStatusChanged:" + statusCode);
        }
        #endregion
    }
}
~~~~
