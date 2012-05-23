---
layout: article
title: What's in Photon 3
categories: [photon-cloud, photon-server, references]
tags: [overview, quickstart]
---
{% include globals %}


### New
<p>
<strong> 1. High Performance S2S Api (native/c++)</strong><br>
A while back we introduced the TCPClient a managed class better
suited for server to server communication than the standard client
library. The TCPClient had two disadvantages: first the programming
model on both ends of the connection was different (TCPClient on one
side, Peer on the other), second it was designed for low-bandwidth
communication. With this release photon can setup a connection to
other photon instances leveraging the networking power of the photon
core while allowing to use the same programming model using peers on
both ends of the connection. 
</p>

2.  **Load Balancing**

    With Photon load balancing developers are enabled to create scalable
    multi-node solutions. It includes out of the box a master server
    hosting the lobby and game servers to host the active games. The
    lobby application is delivered with a set of matchmaking methods and
    a load balancing component that distributes the load (where to
    create new games).

3.  **Lite: Event Caching**

    So far, Lite had "Properties" for rooms and actors to store values
    for players who join later on. They are a nice idea but a bit clumsy
    to use: Whenever something has to be stored, you had to set
    properties. In all other cases, you send events.

    The new Lite application combines both and gives you caching for
    events. Per player and event, you could cache the values in it. If
    anyone joins, the events are sent immediately, like they were just
    raised.

4.  **Extended Data Type Support**

    Photon now supports more data-types than ever: Null values,
    Dictionaries, arrays of objects (each with it's own type) and even
    custom classes can be integrated and used in events.

5.  The Peer has new properties NetworkProtocol, LocalIP, LocalPort.
6.  Up to now you configured a global default application. Now it is
    also possible to define a default application per listener.

### Changed

1.  **Lite and Lite Lobby Code-Cleanup**

    Event-, Operation- and Parameter-Codes are cleaned up. Everything
    was reassigned and the predefined codes should be out of your way
    now. There is an extra page which lists each old and new value.

2.  **Type of Codes**

    We decided to keep using bytes as type for codes. It's just
    effective and lean. If needed, any operation or event could get
    sub-codes (not supported out of the box).

    The ReturnCode for Operation responses was of type int. This is now
    short-typed. Not extremely lean. But short.

3.  **Protocol Cleanup**

    We cleaned up the protocol for Photon 3. This is next to invisible
    for you, but has some benefits. Most of all, the new protocol does
    no longer hard-code any codes. Any code for operations, results,
    parameters or events can be defined by your code.

4.  **Error Handling**

    The error handling was re-designed to

    -   always call the unhandled exception event handler
    -   it is no longer required to initialize the logging facade
    -   introduced three policies for unhandled exceptions: ignore,
        restart application, end process
    -   consistent behavior: before unhandled exceptions in threads
        controlled by photon where logged and then ignored, those in
        other threads usually terminated the process. Note: due to a bug
        in 2.6 sometimes exceptions where "transformed" into
        ThreadAbortException or the application was unloaded without
        being loaded again (this is also fixed with 2.6.28).

5.  **Server SDK Changes (breaking)**

    -   Peer classes are now required to inherit from class 'PeerBase'.
        The former interface IPeer and class PhotonPeer were removed.
        Take a thorough look at the PeerBase class definition as it is a
        central piece of your application.
    -   Applications are now required to inherit from class
        'ApplicationBase'. Like PeerBase this is one of the most
        important classes.
    -   Classes ResponseParameterAttribute, RequestParameterAttribute
        and EventParameterAttribute have been substituted with class
        'DataMemberAttribute'. As a consequence request and response
        parameters cannot be part of the same container class anymore.
    -   The Params property of classes EventData, OperationRequest and
        OperatioResponse has been substituted wth the new property
        'Parameters'. This dictionary's keys are now of type byte.
    -   The property EventData.EventCode has been renamed to 'Code' and
        its type was changed to byte.
    -   The type of the properties OperationRequest.OperationCode and
        OperationResponse.OperationCode has been changed to byte.
    -   The new class 'DataContract' substitutes classes Operation and
        Event. While class Operation remains part of the framework it is
        not mandatory to use it in order to convert the parameters to
        class properties anymore.
    -   Methods Event.GetEventData and Operation.GetOperationResponse
        have been removed. Instead use the new OperationResponse and
        EventData constructors.
    -   PhotonPeer.SendBufferFull was removed. Override
        PeerBase.OnSendBufferFull to respond to a full send buffer. The
        counterpart PeerBase.OnSendBufferEmpty is called when Photon is
        ready to send more data.

6.  **Client SDK Changes (breaking)**

    -   renamed IPhotonListener interface member OperationResult to
        OnOperationResponse, EventAcion to OnEvent and
        PeerStatusCallback to OnStatusChanged.
    -   The different parameters of the old OperationResult
        (OperationCode, Paramters, ...) are passed in one object
        OperationResponse. Same applies to OnEvent now has one parameter
        EventData.
    -   Support for invocationId was removed.

7.  PhotonHostRuntime version attribute was removed from the config file
    and is no longer required - we recommend to remove it to ease
    feature updates.
8.  The application NetSyncObjects is no longer part of the SDK.
9.  Changed default for dump files from mini to full
10. Added version display to log file during start up.

### Bug Fixes

1.  Fixed a memory leak when canceling actions scheduled on a fiber.
2.  S2S.TCPClient did not close the socket at disconnect which caused an
    error when calling Connect again.
3.  Flash policy file is claimed malformed with a top-level XML tag that
    is not , see
    http://www.adobe.com/devnet/flashplayer/articles/fplayer9\_security.html\#\_Malformed\_Policy\_Files
4.  Fixed CLR startup to ensure GC Server is used (issue affected .net
    4.0 only).
5.  Fixed ENet fragmented buffering flow control bug.
6.  Fixed TCP magic byte issues - fragmented pings where not parsed
    correctly and an issue in the internal handling of the size of
    packets could lead to parsing errors (bigger packets and high load).
7.  Fixed Photon hang on shutdown trying to log on a dead logging
    instance.
8.  Fixed the method Peer GetRemoteAddress() - due to a caching issue
    the result was wrong in some cases.
9.  Fixed mini dumps - now obey configured limits.
10. Changed PhotonPeer::GetRemotePort() so that it doesn't rely on a
    cached port.

