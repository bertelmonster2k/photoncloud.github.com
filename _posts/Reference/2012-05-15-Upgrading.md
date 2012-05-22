---
layout: article
title: Upgrading Lite 2.6 to 3.0
categories: [photon-server, references]
tags: [sample, how-to, setup, installation, lite]
---
{% include globals %}

## Lite Project Upgrade Example

*Upgrade your project from Lite 2.6 to 3.0* 

1.  Open the Lite 3.0 solution
2.  Add your old project to the solution and update the references
3.  Compile:
<figure>
<img src="{{ IMG }}/UpgradeLobby1.png" />
</figure>
    
4.  Open class 'LiteLobbyApplication'

    Old code:
<figure>
<img src="{{ IMG }}/UpgradeLobby3.png" />
</figure>

    New Code:

    ~~~~ {.code}
    protected override PeerBase CreatePeer(InitRequest initRequest)
    {
        return new LiteLobbyPeer(
                    initRequest.Protocol, 
                    initRequest.PhotonPeer);
    }
    ~~~~

    What changed:

    -   The signature of 'CreatePeer' has changed.
    -   All peers have to inherit from PeerBase.
    -   IPeer and PhotonPeer have been removed.

    

5.  Open class 'LiteLobbyPeer':
<figure>
<img src="{{ IMG }}/UpgradeLobby4.png" />
</figure>


    Problems:

    -   The LitePeer constructor has changed
    -   The signature of OnOperationRequest has changed
    -   The OperationRequestDispatcher has been merged with LitePeer
    -   PublishOperationResponse has been renamed to
        SendOperationResponse
    -   The OperationRequest constructor has changed


    Solution Steps:

    -   Adjust the contructor signature:

        ~~~~ {.code}
         public LiteLobbyPeer(
            IRpcProtocol rpcProtocol, IPhotonPeer nativePeer) 
            : base(rpcProtocol, nativePeer) { }
        ~~~~

    -   Remove the OnOperationRequest override.
    -   Move the OperationRequestDispatcher methods to LiteLobbyPeer.
    -   Remove the OperationRequestDispatcher field.
    -   Remove the OperationRequestDispatcher class.


    Old code:
<figure>
<img src="{{ IMG }}/UpgradeLobby5.png" />
</figure>


    New code:

    ~~~~ {.code}
    protected override void HandleJoinOperation(
        OperationRequest operationRequest, 
        SendParameters sendParameters)
    {
        var joinOperation = new JoinOperation(
                                this.Protocol, 
                                operationRequest);

        if (!this.ValidateOperation(
                joinOperation, 
                sendParameters))
        {
            return;
        }

        if (joinOperation.GameId.EndsWith(LobbySuffix))
        {
            this.HandleJoinLobby(
                joinOperation, 
                sendParameters);
        }
        else if (string.IsNullOrEmpty(joinOperation.LobbyId) == false)
        {
            this.HandleJoinGameWithLobby(
                joinOperation, 
                sendParameters);
        }
        else
        {
            base.HandleJoinOperation(
                operationRequest, 
                sendParameters);
        }
    }
    ~~~~

    What changed:

    -   Send parameters like reliability, channelId, etc moved from
        OperationRequest/Response to the new struct SendParameters.


    Old code: ![](../img/UpgradeLobby6.png)
<figure>
<img src="{{ IMG }}/UpgradeLobby6.png" />
</figure>

    New code:

    ~~~~ {.code}
    protected virtual void HandleJoinLobby(
        JoinOperation joinOperation, 
        SendParameters sendParameters)
    {
        this.RemovePeerFromCurrentRoom();

        RoomReference lobbyReference = 
            LiteLobbyRoomCache.Instance.GetRoomReference(
                joinOperation.GameId);

        this.RoomReference = lobbyReference;

        lobbyReference.Room.EnqueueOperation(
            this, 
            joinOperation.OperationRequest, 
            sendParameters);
    }
    ~~~~

    What changed:

    -   LitePeer.State has been renamed to 'RoomReference'.


    Old code:
<figure>
<img src="{{ IMG }}/UpgradeLobby7.png" />
</figure>

    New code:

    ~~~~ {.code}
    protected virtual void HandleJoinGameWithLobby(
        JoinOperation joinOperation, 
        SendParameters sendParameters)
    {
        this.RemovePeerFromCurrentRoom();

        RoomReference gameReference = 
            LiteLobbyGameCache.Instance.GetRoomReference(
                joinOperation.GameId, 
                joinOperation.LobbyId);
        
        this.RoomReference = gameReference;
        
        gameReference.Room.EnqueueOperation(
            this, 
            joinOperation.OperationRequest, 
            sendParameters);
    }
    ~~~~

6.  Open class LiteLobby.Operations.JoinOperation:

    Old code:
<figure>
<img src="{{ IMG }}/UpgradeLobby2.png" />
</figure>

    New code:

    ~~~~ {.code}
    public class JoinOperation : Lite.Operations.JoinRequest
    {
        public JoinOperation(
            IRpcProtocol protocol,
            OperationRequest operationRequest)
            : base(protocol, operationRequest)
        {
        }

        [DataMember(
            Code = (byte)LobbyParameterKeys.LobbyId, 
            IsOptional = true)]
        public string LobbyId { get; set; }
    }
    ~~~~

    What changed:

    -   Contracts for operation requests and responses have been devided
        into separate classes. In this instance the new contracts are
        'JoinRequest' and 'JoinResponse'.
    -   The attributes RequestParameter, ResponseParameter and
        EventParameter have been replaced with 'DataMemberAttribute'.
    -   Operations have two new constructors, one for incoming messages
        and one for outgoing messages. Requests need to use the one for
        incoming messages.


7.  Open class 'LiteLobbyGame'

    Old code:
 <figure>
<img src="{{ IMG }}/UpgradeLobby8.png" />
</figure>

    New code:

    ~~~~ {.code}
    protected override Actor HandleJoinOperation(
        LitePeer peer, 
        JoinRequest joinOperation, 
        SendParameters sendParameters)
    {
        Actor actor = base.HandleJoinOperation(
                            peer, 
                            joinOperation, 
                            sendParameters);
    ~~~~

    What changed:

    -   HandeJoinOperation signature has changed


8.  Open class 'LiteLobbyRoom'

    Old code: ![](../img/UpgradeLobby9.png)
  <figure>
<img src="{{ IMG }}/UpgradeLobby9.png" />
</figure>

    New code:

    ~~~~ {.code}
    protected override void ExecuteOperation(
        LitePeer peer, 
        OperationRequest operationRequest, 
        SendParameters sendParameters)
    {
        switch ((OperationCode)operationRequest.OperationCode)
        {
            case OperationCode.Join:
                var joinOperation = new JoinOperation(
                                        peer.Protocol, 
                                        operationRequest);
                
                if (peer.ValidateOperation(
                    joinOperation, 
                    sendParameters) == false)
                {
                    return;
                }

                this.HandleJoinOperation(
                    peer, 
                    joinOperation,sendParameters);
                return;
        }

        base.ExecuteOperation(
            peer, 
            operationRequest, 
            sendParameters);
    }
    ~~~~

    What changed:

    -   ExecuteOperation signature has changed
    -   JoinOperation constructor requires the used protocol
    -   ValidateOperation requires the used send parameters for a
        potential error response


    Old code:
<figure>
<img src="{{ IMG }}/UpgradeLobby10.png" />
</figure>

    New code:

    ~~~~ {.code}
    protected virtual Actor HandleJoinOperation(
        LitePeer peer, 
        JoinOperation joinOperation, 
        SendParameters sendParameters)
            {
                Actor actor = base.HandleJoinOperation(
                                    peer, 
                                    joinOperation, 
                                    sendParameters);
    ~~~~

    What changed:

    -   HandleJoinOperation requires the SendParameters


    Old code:
<figure>
<img src="{{ IMG }}/UpgradeLobby11.png" />
</figure>

    New code:

    ~~~~ {.code}
    protected override void PublishJoinEvent(
        LitePeer peer, 
        Lite.Operations.JoinRequest joinOperation)
    ~~~~

    What changed:

    -   Lite.Operations.JoinOperation has been renamed to 'JoinRequest'


    Old code:
<figure>
<img src="{{ IMG }}/UpgradeLobby12.png" />
</figure>

    New code:

    ~~~~ {.code}
    protected override void PublishLeaveEvent(
        LitePeer peer, 
        LeaveRequest leaveOperation)
    ~~~~

    What changed:

    -   Lite.Operations.LeaveOperation has been renamed to
        'LeaveRequest'


    Old code:
<figure>
<img src="{{ IMG }}/UpgradeLobby13.png" />
</figure>

    New code:

    ~~~~ {.code}
    Log.WarnFormat("RemovePeerFromGame - Actor to remove not found for peer: {0}", peer.ConnectionId);
    ~~~~

    What changed:

    -   LitePeer.Sessionid has been replaced with PeerBase.ConnectionId.


    Old code:
<figure>
<img src="{{ IMG }}/UpgradeLobby14.png" />
</figure>

    New code:

    ~~~~ {.code}
    this.PublishEvent(
        customEvent, 
        this.Actors, 
        new SendParameters { Unreliable = true });
    ~~~~

    What changed:

    -   The flags for reliability, flush, etc moved to struct
        SendParameters


    Old code:
<figure>
<img src="{{ IMG }}/UpgradeLobby15.png" />
</figure>

    New code:

    ~~~~ {.code}
    this.PublishEvent(
        customEvent, 
        this.Actors, 
        new SendParameters { Unreliable = true });
    ~~~~

Congratulations: The Photon 3.0 upgrade is now complete.
