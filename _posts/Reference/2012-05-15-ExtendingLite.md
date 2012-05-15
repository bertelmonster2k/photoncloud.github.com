---
layout: article
title: Extending Lite
categories: [photon-server, references]
tags: [sample, lite, lobby]
---
{% include globals %}

Persisting Data
---------------

Persistency is currently not covered in the SDKs we provide. None of our
applications saves any data. Every game and application is different and
on a high performance server solution, you probably want to control this
aspect yourself.

You are free to use any of the many professional solutions developed in
C\#. As example, take a look at:

-   [NHibernate](http://nhforge.org): mature, open source
    object-relational mapper for the .NET framework
-   [Membase](http://www.membase.org): distributed key-value database
    management system
-   [CSharp-SQLite](http://code.google.com/p/csharp-sqlite): sql
    database in a local file
-   [Lightspeed](http://www.mindscapehq.com/products/lightspeed): high
    performance .NET domain modeling and O/R mapping framework

Trigger game-logic in intervals
-------------------------------

If you want your server application to execute some logic in intervals,
add this as method into LiteGame and schedule a message for the room. By
using a message, the method call will be in-sequence with any operations
(avoiding threading issues).

In the Lite Lobby application's LiteLobbyRoom.cs, we used this code:

~~~~ {.code}
    
    /// Schedules a broadcast of all changes.
    private void SchedulePublishChanges()
    {
        var message = new RoomMessage((byte)LobbyMessageCode.PublishChangeList);
        this.schedule = this.ScheduleMessage(message, LobbySettings.Default.LobbyUpdateIntervalMs);
    }
    
    /// Initializes a new instance of the LiteLobbyRoom class.
    public LiteLobbyRoom(string lobbyName)
        : base(lobbyName)
    {
        this.roomList = new Hashtable();
        this.changedRoomList = new Hashtable();

        // schedule sending the change list
        this.SchedulePublishChanges();
    }

    /// Sends the change list to all users in the lobby and then clears it.
    private void PublishChangeList()
    {
        //do something in intervals...

        //schedule the next call!
        this.SchedulePublishChanges();
    }
    
~~~~

As you can see, at the end of PublishChangeList(), the the next call is
scheduled explicitly.
