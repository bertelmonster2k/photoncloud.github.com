---
layout: article_cloud
title: Loadbalancing Demo
categories: [photon-cloud, tutorials, getting-started]
tags: [how-to, quickstart]
---

<p>
Photon is a light-weight and sophisticated way to use middleware to free you from all the hassle of programming network code. Common operations and methods already implemented within the Photon middleware will give you the time to focus on gameplay logic and the more interesting aspects of turning your idea into a success. See for yourself!
</p>
<p>
In this post we'll show you some chosen excerpts from our Photon Cloud "Loadbalancing Demo". The demo that comes with our SDK shows you how to implement some of the typical use cases you will face when trying to add multiplayer capabilities to your application. In the following we will take a look at the basic Photon specific operations and how they are used in the context of this demo. Easy readable C# code samples will be used in order to illustrate what Photon enables you to do right out of the box.
</p>                 
    			
<h2>Connect To Master Server</h2>
<p><!-- prevent sub-headline styles h*+h* -->
The first thing you need to do is to connect your client to the cloud. We call this process connecting to the Master Server. From there on the Master Server will take care of all the clients' transfers to game servers, load-balancing within the cloud and the available rooms.
</p>
<p>
Let's delve right into the code of our demo:
</p>
```
public DemoClient(): base()
{
  this.PlayerName = "Player" + this.randomNumber.Next(1, 100);
  this.ConnectToMaster("yourAppIdHere", 
                       "v0.1", 
                       PlayerName); 
}
```
Let's take a closer look at the arguments used in the "ConnectToMaster" operation:

1. The Application ID identifies your application within our cloud system. If you don't already have an Application ID you can get it <a href="https://www.exitgames.com/Download/Photon">here</a> for free. You need to insert a valid Application ID in order for this demo to work.
2. The second parameter is the version number. The version number is a string that you can choose by yourself - it is used to make sure that only clients with the same version number get matched and can communicate with each other. This allows you to have different builds of your application online and playable!
3. The third parameter is the name of the player, we won't get into the rocket science involved in this complex construct. Suffice to say it is used by other players to identify the person playing.

After you are connected, the Photon Cloud is ready to do your bidding. You don't have to micromanage from here on, everything is now handled for you! In the diagram below you can see that everything in the grey square is organized by Photon Cloud. The client only needs to send simple operations, like the ones next to the arrows.  			


