---
layout: article
title: Boot Camp Demo: Prediction
categories: [photon-cloud, tutorials]
tags: [sample, how-to, unity, demo]
---
{% include globals %}

Bootcamp Multiplayer Demo
=========================

Prediction
----------

Events arrive delayed because of network latency and maybe also because
of processing delays due to low frame rate or high server load. To
minimize a position synchronization delay we predict where the remote
players might have moved to when we receive a new movement update. For
our prediction algorithm we assume that the remote player kept moving
with the same speed into the same direction. Rotation is excluded from
our prediction.

~~~~ {.code}

/// From file: PlayerRemote.cs
    
    /// updates a (remote player's) position. directly gets the new position from the received event
   internal void SetPosition(Hashtable evData)
   {

    Vector3 p = GetPosition((float[])evData[Constants.STATUS_PLAYER_POS]);
    Quaternion r = GetRotation((float[])evData[Constants.STATUS_PLAYER_ROT]);
    float diff =  UnityEngine.Time.time - this.lastUpdateTime;
    if (diff > 0.2f)
    {
          /// old info, walk to newest available (skip prediction)
       this.pos = p;    
    }
    else 
    {
          /// predict that he continues to walk into same direction with same speed
           this.pos = p + p - realPos;              
    }


    this.rot = r;   
    this.realPos = p;
    this.lastUpdateTime = UnityEngine.Time.time;

    targetpos = GetPosition((float[])evData[Constants.STATUS_TARGET_POS]);
    SendMessage("SetTargetPos", targetpos);           
   }

    
  void Update()
  {
       if (this.lastUpdateTime > 1f)
    {
        /// move to real pos in case we have not received anything for a second
        this.pos = this.realPos;
        this.lastUpdateTime = UnityEngine.Time.time;
    }

  }
~~~~
