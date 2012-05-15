---
layout: article
title: Overview PUN
categories: [photon-cloud, tutorials]
tags: [unity, how-to, sample, overview]
---
{% include globals %}

### PhotonNetwork Plugin for Unity3d

This plugin consists of quite a few files, however there is only one
that truly matters: PhotonNetwork. This class contains all functions and
variables that you need. If you ever have custom requirements, you can
always modify the source files - this plugin is just an implementation
of Photon after all.

The imported package includes a setup wizard, which creates a
configuration for either the cloud service or your own Photon server.
Check: PhotonServerSettings.

Using Unity Javascript? To be able to use the Photon classes you'll need
to move the Plugins folder to the root of your project.

To show you how this API works, here are a few examples right away.

### Connecting to games

PhotonNetwork always uses a master server and one or more game servers.
The master server manages the list of game servers and currently running
games on those servers. To pick a game (or get into a random one),
players connect to the Master server. The Master forwards the clients to
the game servers, where the actual gameplay is done. The servers are all
run on dedicated machines - there is no such thing as player-hosted
'servers'. You don't have to bother remembering about the server
organization though, as the API all hides this for you.

~~~~ {.code}

PhotonNetwork. ConnectUsingSettings("v1.0"); 
~~~~

#### Versioning

The loadbalancing logic for Photon uses your appID to separate your
players from anyone else's. The same is done by game version, which
separates players with a new client from those with older clients. As we
can't guarantee that different Photon Unity Networking versions are
compatible with each other, we add the PUN version to your game's
version before sending it (since PUN v1.7).

### Creating and joining games

Next, you'll want to join or create a room. The following code showcases
some required functions:

~~~~ {.code}

//Join a room 
PhotonNetwork.JoinRoom(roomName);  
 
//Create this room.  
PhotonNetwork.CreateRoom(roomName);  
// Fails if it already exists and calls: OnPhotonCreateGameFailed 
 
//Tries to join any random game: 
PhotonNetwork.JoinRandomRoom();  
//Fails if there are no matching games: OnPhotonRandomJoinFailed 
~~~~

A list of currently running games is provided by the master server's
lobby. It can be joined like other rooms but only provides and updates
the list of rooms. The PhotonNetwork plugin will automatically join the
lobby after connecting. When you're joining a room, the list will no
longer update.

To display the list of rooms (in a lobby):

~~~~ {.code}

foreach (Room game in PhotonNetwork.GetRoomList()) 
{ 
    GUILayout.Label(game.name + " " + game.playerCount + "/" + game.maxPlayers); 
} 
~~~~

Alternatively, the game can use random matchmaking: It will try to join
any room and fail if none has room for another player. In that case:
Create a room without name and wait until other players join it
randomly.

### MonoBehaviour Callbacks

PhotonNetwork implements several callbacks to let your game know about
state changes, like "connected" or "joined a game". Each of the methods
used as callback is part of the PhotonNetworkingMessage enum. Per enum
item, the use is explained (check the tooltip when you type in e.g.
PhotonNetworkingMessage. OnConnectedToPhoton. You can add these methods
on any number of MonoBehaviours, they will be called in the respective
situation. The complete list of callbacks is also in the [Plugin
reference](#_PluginReference).

This covers the basics of setting up game rooms. Next up is actual
communication in games.

### Sending messages in game rooms

Inside a room you are able to send network messages to other connected
players. Furthermore you are able to send buffered messages that will
also be sent to players that connect in the future (for spawning your
player for instance).

Sending messages can be done using two methods. Either RPCs or by using
the PhotonView property OnSerializePhotonView. There is more network
interaction though. You can listen for callbacks for certain network
events (e.g. OnPhotonInstantiate, OnPhotonPlayerConnected) and you can
trigger some of these events (PhotonNetwork.Instantiate). Don't worry if
you're confused by the last paragraph, next up we'll explain for each of
these subjects.

### PhotonView

PhotonView is a script component that is used to send messages (RPCs and
OnSerializePhotonView). You need to attach the PhotonView to your games
gameobjects. Note that the PhotonView is very similar to Unity's
NetworkView.

At all times, you need at least one PhotonView in your game in order to
send messages and optionally instantiate/allocate other PhotonViews.

To add a PhotonView to a gameobject, simply select a gameobject and use:
"Components/Miscellaneous/Photon View".

![](../img/PhotonView.PNG)

#### Observe Transform

If you attach a Transform to a PhotonView's Observe property, you can
choose to sync Position, Rotation and Scale or a combination of those
across the players. This can be a great help for prototyping or smaller
games. Note: A change to any observed value will send out all observed
values - not just the single value that's changed. Also, updates are not
smoothed or interpolated.

#### Observe MonoBehaviour

A PhotonView can be set to observe a MonoBehaviour. In this case, the
script's OnPhotonSerializeView method will be called. This method is
called for writing an object's state and for reading it, depending on
whether the script is controlled by the local player.

The simple code below shows how to add character state synchronization
with just a few lines of code more:

~~~~ {.code}

void OnPhotonSerializeView(PhotonStream stream, PhotonMessageInfo info) 
{ 
   if (stream.isWriting) 
   { 
       //We own this player: send the others our data 
       stream.SendNext((int)controllerScript._characterState); 
       stream.SendNext(transform.position); 
       stream.SendNext(transform.rotation); 
   } 
   else 
   { 
       //Network player, receive data 
       controllerScript._characterState = (CharacterState)(int)stream.ReceiveNext(); 
       correctPlayerPos = (Vector3)stream.ReceiveNext(); 
       correctPlayerRot = (Quaternion)stream.ReceiveNext(); 
   } 
}
~~~~

Now on, to yet another way to communicate: RPCs.

### Remote Procedure Calls

Remote Procedure Calls (RPCs) are exactly what the name implies: methods
that can be called by any client within a room. To enable remote calls
for a method of a MonoBehaviour, you must apply the attribute: [RPC]. A
PhotonView instance is needed on the same GameObject, to call the marked
function.

~~~~ {.code}

[RPC] 
void ChatMessage(string a, string b) 
{ 
    Debug.Log("ChatMessage " + a + " " + b); 
}
~~~~

To call the method from any script, you need access to a PhotonView
object. If your script derives from Photon.MonoBehaviour, it has a
photonView field. Any regular MonoBehaviour or GameObject can use:
PhotonView.Get(this) to get access to its PhotonView component and then
call RPCs on it.

~~~~ {.code}

PhotonView photonView = PhotonView.Get(this); 
photonView.RPC("ChatMessage", PhotonTargets.All, "jup", "and jup!"); 
~~~~

So, instead of directly calling the target method, you call RPC() on a
PhotonView. Provide the name of the method to call, which players should
call the method and then provide a list of parameters.

Careful: The parameters list used in RPC() has to match the number of
expected parameters! If the receiving client can't find a matching
method, it will log an error. There is one exception to this rule: The
last parameter of a RPC method can be of type PhotonMessageInfo, which
will provide some context for each call.

~~~~ {.code}

[RPC] 
void ChatMessage(string a, string b, PhotonMessageInfo info) 
{ 
    Debug.Log(String.Format("Info: {0} {1} {2}", info.sender, info.photonView, 
info.timestamp)); 
}
~~~~
