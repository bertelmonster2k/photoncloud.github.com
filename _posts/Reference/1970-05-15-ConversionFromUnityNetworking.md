---
layout: article
title: Conversion From Unity Networking
categories: [photon-cloud, references]
tags: [sample, how-to, unity]
---
{% include globals %}


## Converting your Unity networking project to Photon

Converting your Unity networking project to Photon can be done in one
day. Just to be sure, make a backup of your project, as our automated
converter will change your scripts. After this is done, run the
converter from the Photon editor window (Window -\> Photon Unity
Networking -\> Converter -\> Start). The automatic conversion takes
between 30 seconds to 10 minutes, depending on the size of your project
and your computers performance. This automatic conversion takes care of
the following:

All NetworkViews are replaced with PhotonViews and the exact same
settings. This is applied for all scenes and all prefabs. This should
work flawlessly.

All scripts (JS/BOO/C\#) are scanned for Network API calls, and they are
replaced with PhotonNetwork calls.

There are some minor differences, therefore you will need to manually
fix a few script conversion bugs. After conversion, you will most likely
see some compile errors. You'll have to fix these first. Most common
compile errors:

<figure>
<img src="{{ IMG }}/CompileErrors.PNG" />
</figure>


You should be able to fix all compile errors in 5-30 minutes. Most
errors will originate from main menu/GUI code, related to
IPs/Ports/Lobby GUI.

This is where Photon differs most from Unity's solution:

There is only one Photon server and you connect using the room names.
Therefore all references to IPs/ports can be removed from your code
(usually GUI code). PhotonNetwork.JoinRoom(string room) only takes a
room argument, you'll need to remove your old IP/port/NAT arguments. If
you have been using the "Ultimate Unity networking project" by M2H, you
should remove the MultiplayerFunctions class.

Lastly, all old MasterServer calls can be removed. You never need to
register servers, and fetching the room list is as easy as calling
PhotonNetwork.GetRoomList(). This list is always up to date (no need to
fetch/poll etc). Rewriting the room listing can be most work, if your
GUI needs to be redone, it might be simpler to write the GUI from
scratch.
