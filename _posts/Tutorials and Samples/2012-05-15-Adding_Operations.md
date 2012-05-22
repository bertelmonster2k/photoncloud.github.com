---
layout: article
title: Adding Operations
categories: [tutorials]
tags: [lite, sample, how-to]
---
{% include globals %}


In many cases, just sending events is not enough for a game. If you want
to provide authorization, persistency or game-specific Operations, it's
time to extend Lite (or LiteLobby).

This page shows two ways to implement new Operations and how to use
them. Our sample Operation expects a message from the client, checks it
and provides return values.

### Server-Side, Simple Variant

Let's start on the server side, in the Lite Application. This variant is
not very elegant but simple.

~~~~ {.code}
    
        //Server SDK, LitePeer.cs
        public void OnOperationRequest(OperationRequest request) 
        { 
            // handle operation here (check request.OperationCode) 
            switch (request.OperationCode) 
            { 
                 case 1: 
               { 
                  var message = (string)request.Params[100]; 
                  if (message == "Hello World") 
                  { 
                     // received hello world, send an answer! 
                     var response = new OperationResponse(request, 0, "OK", new Dictionary<short, object> { { 100, "Hello yourself!" } } ); 
                     this.PhotonPeer.SendOperationResponse(response); 
                  } 
                  else 
                  { 
                     // received something else, send an error 
                     var response = new OperationResponse(request, 1, "Don't understand, what are you saying?"); 
                     this.PhotonPeer.SendOperationResponse(response); 
                  } 
 
                  break; 
               } 
            } 
        } 
    
~~~~

The above code first checks the called OperationCode. In this case, it's
1. In Photon, the OperationCode is a shortcut for the name/type of an
Operation. We'll see how the client call this below.

If Operation 1 is called, we check the request parameters. Here,
parameter 100 is expected to be a string. Again, Photon only uses
byte-typed parameters, to keep things lean during transfer.

The code then checks the message content and prepares a response. Both
responses have a returnCode and a debug message. The returnCode 0 is a
positive return, any other number (here: 1) could mean an error and
needs to be handled by the client.

Our positive response also includes a return value. You could add more
key-value pairs, but here we stick to 100, which is a string.

This already is a complete implementation of an Operation. This is our
convention for a new Operation, which must be carried over to the client
to be used.

### Client Side

Knowing the definition of above's Operation, we can call it from the
client side. This client code calls it on connect:

~~~~ {.code}
    
        public void PeerStatusCallback(int returnCode) 
        { 
            // handle peer status callback 
            switch ((ReturnCode)returnCode) 
            { 
            case ReturnCode.Connect: 
                // send hello world when connected 
                var parameter = new Hashtable { { (byte)100, "Hello World" } }; 
                this.Peer.OpCustom(1, parameter, true); 
                break; 
                //[...]
    
~~~~

As defined above, we always expect a message as parameter 100. This is
provided as parameter Hashtable. We make sure that the parameter key is
not mistaken for anything but a byte, or else it won't send. And we put
in: "Hello World".

The PhotonPeer (and LitePeer) method OpCustom expects the OperationCode
as first parameter. This is 1. The parameters follow and we want to make
sure the operation arrives (it's essential after all).

Aside from this, the client just has to do the usual work, which means
it has to call PhotonPeer.Service in intervals.

Once the result is available, it can be handled like this:

~~~~ {.code}
    
        public void OperationResult(byte opCode, int returnCode, Hashtable returnValues, short invocID) 
        { 
            // handle operation response 
            switch (opCode) 
            { 
                // hello world response 
                case 1: 
                    // OK 
                    if (returnCode == 0) 
                    { 
                        // show the response message 
                        Console.WriteLine(returnValues[(byte)100]); 
                    } 
                    else 
                    { 
                        // show the error message (always param code 1) 
                        Console.WriteLine(returnValues[(byte)1]); 
                    } 
                    break; 
            } 
        } 
    
~~~~

### Server Side, Advanced Version

The preferred way to handle Operations would be to create a class for
it. This way it becomes strongly typed and issues due to missing
parameters are handled by the framework.

This is the definition of the Operation, its parameters, their types and
of the return values:

~~~~ {.code}
    
        //new Operation Class
        namespace MyPhotonServer 
        { 
            using Photon.SocketServer; 
            using Photon.SocketServer.Rpc; 

            public class MyCustomOperation : Operation 
            { 
                public MyCustomOperation(OperationRequest request) : base(request) 
                { 
                } 

                [RequestParameter(Code = 100, IsOptional = false)] 
                [ResponseParameter(Code = 100, IsOptional = false)] 
                public string Message { get; set; } 
            } 
        } 
    
~~~~

With the Operation class defined above, we can map the request to it and
also provide the response in a strongly typed manner.

~~~~ {.code}
    
    public void OnOperationRequest(OperationRequest request)
    {
        switch (request.OperationCode)
        {
            case 1:
            {
                var operation = new MyCustomOperation(request);
                if (peer.ValidateOperation(operation) == false)
                {
                    // received garbage, send an error
                    var response = new OperationResponse(request, 1, "That's garbage!");
                    this.PhotonPeer.SendOperationResponse(response);
                    return;
                }

                if (operation.Message == "Hello World")
                {
                    // received hello world, send an answer!
                    operation.Message = "Hello yourself!";
                    OperationResponse response = operation.GetOperationResponse(0, "OK");
                    this.PhotonPeer.SendOperationResponse(response);
                }
                else
                {
                    // received something else, send an error
                    var response = new OperationResponse(request, 1, "Don't understand, what are you saying?");
                    this.PhotonPeer.SendOperationResponse(response);
                }
                break;
            }
        }
    }
    
~~~~
