---
layout: article_server
title: Loadbalancing Demo
categories: [photon-server, getting_started]
tags: [how-to, getting_started, setup, quickstart, installation]
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