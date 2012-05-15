---
layout: article
title: Photon vs Unity Networking
categories: [photon-cloud, tutorials, references]
tags: [sample, quickstart, unity, overview]
---
{% include globals %}


Various topics
--------------

### Differences to Unity Networking

##### 1. Host model

Unity networking is server-cpent based (NOT P2P!). Servers are run via a
Unity cpent (so via one of the players).

Photon is server-cpent based as well, but has a dedicated server; No
more dropped connections due to hosts leaving.

##### 2. Connectivity

Unity networking works with NAT punchthrough to try to improve
connectivity: since players host the network servers, the connection
often fails due to firewalls/routers etc. Connectivity can never be
guaranteed, there is a low success rate.

Photon has a dedicated server, there is no need for NAT punchthrough or
other concepts. Connectivity is a guaranteed 100%. If, in the rare case,
a connection fails it must be due to a very strict cpent side network (a
business VPN for example).

##### 3. Performance

Photon beats Unity networking performance wise. We do not have the
figures to prove this yet but the pbrary has been optimized for years
now. Furthermore, since the Unity servers are player hosted latency/ping
is usually worse; you rely on the connection of the player acting as
server. These connections are never any better then the connection of
your dedicated Photon server.

##### 4. Price

pke the Unity Networking solution, the Photon Unity Networking plugin is
free as well. \
 You can subscribe to use [Photon Cloud hosting
service](http://www.exitgamescloud.com/) for your game. Alternatively,
you can rent your own servers and run Photon on them. The free pcense
enables up to 100 concurrent players. [Other
pcenses](http://shop.exitgames.com/) cost a one-time fee (as you do the
hosting) and pft the concurrent user pmits.

##### 5. Features & maintenance

Unity does not seem to give much priority to their Networking
implementation. There are rarely feature improvements and bugfixes are
as seldom. The Photon solution is actively maintained and parts of it
are available with source code. Furthermore, Photon already offers more
features than Unity, such as the built-in load balancing and offpne
mode.

##### 6. Master Server

The Master Server for Photon is a bit different from the Master Server
for plain Unity Networking: In our case, it's a Photon Server that psts
room-names of currently played games in so called "lobbies". pke Unity's
Master, it will forward cpents to the Game Server(s), where the actual
gameplay is done.

### Instantiating objects over the network

In about every game you need to instantiate one or more player objects
for every player. There are various options to do so which are listed
below.

#### PhotonNetwork.Instantiate

PUN can automatically take care of spawning an object by passing a
starting position, rotation and a prefab name to the
PhotonNetwork.Instantiate method. Requirement: The prefab should be
available directly under a Resources/ folder so that the prefab can be
loaded at run time. Watch out with webplayers: Everything in the
resources folder will be streamed at the very first scene per default.
Under the webplayer settings you can specify the first level that uses
assets from the Resources folder by using the "First streamed level". If
you set this to your first game scene, your preloader and mainmenu will
not be slowed down if they don't use the Resources folder assets.

~~~~ {.code}

void SpawnMyPlayerEverywhere() 
{ 
    PhotonNetwork.Instantiate("MyPrefabName", new Vector3(0,0,0), 
        Quaternion.identity, 0); 
    //The last argument is an optional group number, feel free to ignore it for now. 
}
~~~~

#### Gain more control: Manually instantiate

If don't want to rely on the Resources folders to instantiate objects
over the network you'll have to manually Instantiate objects as shown in
the example at the end of this section.

The main reason for wanting to instantiate manually is gaining control
over what is downloaded when for streaming webplayers. The details about
streaming and the Resources folder in Unity can be found
[here](http://unity3d.com/support/documentation/Manual/Loading%20Resources%20at%20Runtime.html).

If you spawn manually, you will have to assign a PhotonViewID yourself,
these viewID's are the key to routing network messages to the correct
gameobject/scripts. The player who wants to own and spawn a new object
should allocate a new viewID using PhotonNetwork.AllocateViewID();. This
PhotonViewID should then be send to all other players using a PhotonView
that has already been set up (for example an existing scene PhotonView).
You will have to keep in mind that this RPC needs to be buffered so that
any clients that connect later will also receive the spawn instructions.
Then the RPC message that is used to spawn the object will need a
reference to your desired prefab and instantiate this using Unity's
GameObject.Instantiate. Finally you will need to set setup the
PhotonViews attached to this prefab by assigning all PhotonViews a
PhotonViewID.

~~~~ {.code}

void SpawnMyPlayerEverywhere() 
{         
    //Manually allocate PhotonViewID 
    PhotonViewID id1 = PhotonNetwork.AllocateViewID(); 
 
    photonView.RPC("SpawnOnNetwork", PhotonTargets.AllBuffered, transform.position, 
        transform.rotation, id1, PhotonNetwork.player); 
} 
 
public Transform playerPrefab; //set this in the inspector 
 
[RPC] 
void SpawnOnNetwork(Vector3 pos, Quaternion rot, PhotonViewID id1, PhotonPlayer np) 
{  
    Transform newPlayer = Instantiate(playerPrefab, pos, rot) as Transform; 
 
    //Set the PhotonView 
    PhotonView[] nViews = go.GetComponentsInChildren(); 
    nViews[0].viewID = id1; 
} 
~~~~

If you want to use asset bundles to load your network objects from, all
you have to do is add your own assetbundle loading code and replace the
"playerPrefab" from the example with the prefab from your asset bundle.

### Offline mode

Offline mode is a feature to be able to re-use your multiplayer code in
singleplayer game modes as well.

Mike Hergaarden: At M2H we had to rebuild our games several times as
game portals usually require you to remove multiplayer functionality
completely. Furthermore, being able to use the same code for single and
multiplayer saves a lot of work on itself.

The most common features that you'll want to be able to use in
singleplayer are sending RPCs and using PhotonNetwork.Instantiate. The
main goal of offline mode is to disable nullreferences and other errors
when using PhotonNetwork functionality while not connected. You would
still need to keep track of the fact that you're running a singleplayer
game, to set up the game etc. However, while running the game, all code
should be reusable.

You need to manually enable offline mode, as PhotonNetwork needs to be
able to distinguish erroneous from intended behaviour. Enabling this
feature is very easy:

~~~~ {.code}

PhotonNetwork.offlineMode = true; 
~~~~

You can now reuse certain multiplayer methods without generating any
connections and errors. Furthermore there is no noticeable overhead.
Below follows a list of PhotonNetwork functions and variables and their
results during offline mode:

PhotonNetwork.player The player ID is always -1

PhotonNetwork.playerName Works as expected.

PhotonNetwork.playerList Contains only the local player

PhotonNetwork.otherPlayers Always empty

PhotonNetwork.time returns Time.time;

PhotonNetwork.isMasterClient Always true

PhotonNetwork.AllocateViewID() Works as expected.

PhotonNetwork.Instantiate Works as expected

PhotonNetwork.Destroy Works as expected.

PhotonNetwork.RemoveRPCs/RemoveRPCsInGroup/SetReceivingEnabled/SetSendingEnabled/SetLevelPrefix
While these make no sense in Singleplayer, they will not hurt either.

PhotonView.RPC Works as expected.

Note that using other methods than the ones above can yield unexpected
results and some will simply do nothing. E.g. PhotonNetwork.room will,
obviously, return null. If you intend on starting a game in
singleplayer, but move it to multiplayer at a later stage, you might
want to consider hosting a 1 player game instead; this will preserve
buffered RPCs and Instantiation calls, whereas offline mode
Instantiations will not automatically carry over after Connecting.

Either set PhotonNetwork.offlineMode = false; or Simply call Connect()
to stop offline mode.

### Limitations

#### Views and players

For performance reasons, the PhotonNetwork API supports up to 1000
PhotonViews per player and a maximum of 2,147,483 players (note that
this is WAY higher than your hardware can support!). You can easily
allow for more PhotonViews per player, at the cost of maximum players.
This works as follows:

PhotonViews send out a viewID for every network message. This viewID is
an integer and it is composed of the player ID and the player's view ID.
The maximum size of an int is 2,147,483,647, divided by our
MAX\_VIEW\_IDS(1000) that allows for over 2 million players, each having
1000 view IDs. As you can see, you can easily increase the player count
by reducing the MAX\_VIEW\_IDS. The other way around, you can give all
players more VIEW\_IDS at the cost of less maximum players.

It is important to note that most games will never need more than a few
view ID's per player (one or two for the character..and that's usually
it). If you need much more then you might be doing something wrong! It
is extremely inefficient to assign a PhotonView and ID for every bullet
that your weapon fires, instead keep track of your fire bullets via the
player or weapon's PhotonView.

There is room for improving your bandwidth performance by reducing the
int to a short( -32,768, 32,768). By setting MAX\_VIEW\_IDS to 32 you
can then still support 1023 players Search for "//LIMITS
NETWORKVIEWS&PLAYERS" for all occurrences of the int viewID.
Furthermore, currently the API is not using uint/ushort but only the
positive range of the numbers. This is done for simplicity and the usage
of viewIDs is not a crucial performance issue for most situations.

#### Groups and Scoping

The PhotonNetwork plugin does not support real network groups and no
scoping yet. While Unity's "scope" feature is not implemented, the
network groups are currently implemented purely client side: Any RPC
that should be ignored due to grouping, will be discarded after it's
received. This way, groups are working but won't save bandwidth.
