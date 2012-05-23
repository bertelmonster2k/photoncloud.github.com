---
layout: article
title: Updated Values
categories: [photon-cloud, photon-server, references]
tags: [overview, how-to]
---
{% include globals %}


## Updated: Event-, Operation- and Parameter-Codes

In Photon 2.x, our demo applications Lite and Lite Lobby used just about
any (more or less random) value for event-, operation- and
parameter-codes. This easily clashed when our projects are used as basis
and get extended.

For Photon 3.x, we came up with this new rule to assign codes: **Our
codes begin with the max value, then go down.** Example: Operation Join
is now code 255 in Lite. If you want to add a new events (codes) to
Lite, start with 0 and increment as you go.

### Map of old and new codes

The following tables include codes from Lite and Lite Lobby
applications. If you use our named constants for these (which you
should), you don't have to know these values.

<table class="list width100">
  <thead>
	<tr>
	<th> OperationCode </th>
	<th> Old Value </th>
	<th> New Value </th>
	</tr>
	<thead>
	<tbody>
	<tr>
	<td> Join </td>
	<td> 90 </td>

	<td> 255 </td>
	</tr>
	<tr>
	<td> Leave </td>
	<td> 91 </td>
	<td> 254 </td>
	</tr>
	<tr>
	<td> RaiseEvent </td>

	<td> 92 </td>
	<td> 253 </td>
	</tr>
	<tr>
	<td> SetProperties </td>
	<td> 93 </td>
	<td> 252 </td>

	</tr>
	<tr>
	<td> GetProperties </td>
	<td> 94 </td>
	<td> 251 </td>
	</tr>
	<tr>
	<td> EstablishSecureCommunication </td>
	<td> 95 </td>

	<td> (obsolete) </td>
	</tr>
	<tr>
	<td> Ping </td>
	<td> 104 </td>
	<td> (obsolete) </td>
	</tr>
	</tbody>
</table>
</p>
<p>
<table class="list width100">
	<thead>
	<tr>
	<th> EventCode </th>
	<th> Old Value </th>
	<th> New Value </th>
	</tr>
	<thead>
	<tbody>
	<tr>
	<td> Join </td>
	<td> 90 </td>

	<td> 255 </td>
	</tr>
	<tr>
	<td> Leave </td>
	<td> 91 </td>
	<td> 254 </td>
	</tr>
	<tr>
	<td> PropertiesChanged </td>

	<td> 92 </td>
	<td> 253 </td>
	</tr>
	<tr>
	<td> GameList (LiteLobby) </td>
	<td> 1 </td>
	<td> 252 </td>

	</tr>
	<tr>
	<td> GameListUpdate (LiteLobby) </td>
	<td> 2 </td>
	<td> 251 </td>
	</tr>
	</tbody>
</table>
</p>
<p>
<table class="list width100">
	<thead>
	<tr>
	<th> ParameterCode </th>
	<th> Old Value </th>
	<th> New Value </th>
	</tr>
	<thead>
	<tbody>
	<tr>
	<td> ERR </td>
	<td> 0 </td>

	<td> (obsolete) </td>
	</tr>
	<tr>
	<td> DBG </td>
	<td> 1 </td>
	<td> (obsolete) </td>
	</tr>
	<tr>
	<td> GameId </td>

	<td> 4 </td>
	<td> 255 </td>
	</tr>
	<tr>
	<td> ActorNr </td>
	<td> 9 </td>
	<td> 254 </td>

	</tr>
	<tr>
	<td> TargetActorNr </td>
	<td> 10 </td>
	<td> 253 </td>
	</tr>
	<tr>
	<td> Actors </td>
	<td> 11 </td>

	<td> 252 </td>
	</tr>
	<tr>
	<td> Properties </td>
	<td> 12 </td>
	<td> 251 </td>
	</tr>
	<tr>
	<td> Broadcast </td>

	<td> 13 </td>
	<td> 250 </td>
	</tr>
	<tr>
	<td> ActorProperties </td>
	<td> 14 </td>
	<td> 249 </td>

	</tr>
	<tr>
	<td> GameProperties </td>
	<td> 15 </td>
	<td> 248 </td>
	</tr>
	<tr>
	<td> ClientKey </td>
	<td> 16 </td>

	<td> (obsolete) </td>
	</tr>
	<tr>
	<td> ServerKey </td>
	<td> 17 </td>
	<td> (obsolete) </td>
	</tr>
	<tr>
	<td> Cache </td>

	<td> 18 </td>
	<td> 247 </td>
	</tr>
	<tr>
	<td> ReceiverGroup </td>
	<td> 19 </td>
	<td> 246 </td>

	</tr>
	<tr>
	<td> Data </td>
	<td> 42 </td>
	<td> 245 </td>
	</tr>
	<tr>
	<td> Code </td>
	<td> 60 </td>

	<td> 244 </td>
	</tr>
	<tr>
	<td> Flush </td>
	<td> 61 </td>
	<td> 243 </td>
	</tr>
	<tr>
	<td> LobbyId (LiteLobby) </td>

	<td> 5 </td>
	<td> 242 </td>
	</tr>
	</tbody>
</table>