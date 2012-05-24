---
layout: article
title: Boot Camp Demo - Shot Sync
categories: [photon-cloud, tutorials]
tags: [sample, how-to, unity, demo]
---
{% include globals %}


## Shot Sync (shot rotation and target finding)

Firing Synchronization was accomplished by taking the player’s rotation
while aiming, the target’s position as set in playerLocal, the weapon’s
emission point and it’s corresponding finishing contact point (this last
one only for grenade launchers) into consideration.

The actual shot is detected in the Gun.js script which is responsible
for delivering the shot’s values to the Fire function in PlayerLocal.cs.
An OpRaiseEvent is used for this purpose, and messages are broadcast
only if the fire state is different from the last fire state so that no
redundant and traffic consuming messages are broadcast.

~~~~ {.code}

/// From file: Gun.js
    
   ///fire events are broadcast depending on the selected weapon type.
   switch(fireType)
   {
    case FireType.RAYCAST:
      if(Local)
         {
             TrainingStatistics.shootsFired++;
             CheckRaycastHit();
             h = new Hashtable();
             h.Add(1,(cam.ScreenToWorldPoint(
         new Vector3(Screen.width * 0.5, Screen.height * 0.5, cam.farClipPlane))));
             h.Add(2,fire);
             transform.root.SendMessage("Fire", h, 
                                        SendMessageOptions.DontRequireReceiver);
         }
         break;
      
        case FireType.PHYSIC_PROJECTILE:
      if(Local)
         {
             TrainingStatistics.grenadeFired++;
             LaunchProjectile();
             h = new Hashtable();
             h.Add(1,(cam.ScreenToWorldPoint(
         new Vector3(Screen.width * 0.5, Screen.height * 0.55, 40)) ));
             h.Add(2,fire);
             h.Add(3,weaponTransformReference.position);
             transform.root.SendMessage("Fire",h,  
                                         SendMessageOptions.DontRequireReceiver);
           }
           
           fire=false;
           break;
       }
~~~~

~~~~ {.code}

/// From file: PlayerLocal.cs

///fire gets the fire event info packaged in such a way that the server can read it. The info is sent only if the fire state is different from the previous fire state to avoid redundancy and traffic.

    private void Fire(Hashtable _hastable)
    {
       bool fire = false;
       bool reload = true;

       Hashtable evInfo = new Hashtable();

       if (_hastable.Contains("reload"))
       {
            reload = (bool)_hastable["reload"];
            evInfo.Add("reload", reload);
        }

        if (_hastable.Contains(2))
        {
            fire = (bool)_hastable[(int)2];
            evInfo.Add(Constants.STATUS_PLAYER_FIRE, fire);
        }

        if(_hastable.Contains(1))
        {
            Vector3 point = (Vector3)_hastable[(int)1];                    
            evInfo.Add(Constants.STATUS_PLAYER_POINTX, point.x);
            evInfo.Add(Constants.STATUS_PLAYER_POINTY, point.y);
            evInfo.Add(Constants.STATUS_PLAYER_POINTZ, point.z);
        }

        if (_hastable.Contains(3))
        {
            Vector3 outputpoint = (Vector3)_hastable[(int)3];
            evInfo.Add(Constants.STATUS_PLAYER_OUTPUTPOINTX, outputpoint.x);
            evInfo.Add(Constants.STATUS_PLAYER_OUTPUTPOINTY, outputpoint.y);
            evInfo.Add(Constants.STATUS_PLAYER_OUTPUTPOINTZ, outputpoint.z);
        }

        if (reload != oldReload && _hastable.Contains("reload")) 
       {
           num++;
           Debug.Log("reload" + num);
           oldReload = reload;
           engine.peer.OpRaiseEvent(Constants.EV_FIRE, evInfo, true);
       }

        if (fire != oldFire && _hastable.Contains(2))
        {
            numr++;
            Debug.Log("fire" + numr);
            oldFire = fire;

            engine.peer.OpRaiseEvent(Constants.EV_FIRE, evInfo, true);
        }
    }

    private void SetGunInfoRPC(byte Gun)
    {       
        currentWeapon = Gun;               

        Hashtable evInfo = new Hashtable();     
        evInfo.Add("W", (byte)Gun);

        engine.peer.OpSetPropertiesOfActor(this.engine.LocalPlayer.playerID, 
                                           evInfo, true, 0);

    }
~~~~

The message received by Game.cs in its EventAction function executes the
code corresponding to the EV\_FIRE case. This calls SetFire with the
hashtable.

~~~~ {.code}

/// From file: Game.cs
    
   case Constants.EV_FIRE:                  
      p.playerRemote.SetFire((Hashtable)photonEvent[(byte)LiteEventKey.Data]);                    
      break;
~~~~

PlayerRemote.cs contains SetFire; it receives all firing event info,
unpacks it and converts it into UNITY readable data structures. After
that it executes SetFireSimulate (the selected weapon information is
appended here).

~~~~ {.code}

/// From file: PlayerRemote.cs
    
   internal void SetFire(Hashtable evData)
   {
       Hashtable h = new Hashtable();

      if (evData.Contains("reload"))
      {
         this.Reload = (bool)evData["reload"];
      }
      else 
      {
         this.Reload = false;
      }

      this.Fire = (bool)evData[Constants.STATUS_PLAYER_FIRE];
      h.Add("Fire", this.Fire);
      h.Add("Weapon", this.currentWeapon);
      h.Add("reload", this.Reload);
      
      float[] POINT = new float[3];
      POINT[0] = (float)evData[Constants.STATUS_PLAYER_POINTX];
      POINT[1] = (float)evData[Constants.STATUS_PLAYER_POINTY];
      POINT[2] = (float)evData[Constants.STATUS_PLAYER_POINTZ];

      Vector3 point = GetPosition(POINT);
      h.Add("Point", point);

      if (evData.Contains(Constants.STATUS_PLAYER_OUTPUTPOINTX))
      {
         float[] OUTPUTPOINT = new float[3];
         OUTPUTPOINT[0] = (float)evData[Constants.STATUS_PLAYER_OUTPUTPOINTX];
         OUTPUTPOINT[1] = (float)evData[Constants.STATUS_PLAYER_OUTPUTPOINTY];
         OUTPUTPOINT[2] = (float)evData[Constants.STATUS_PLAYER_OUTPUTPOINTZ];
  
         Vector3 outputpoint = GetPosition(OUTPUTPOINT);
         h.Add("OutPutPoint", outputpoint);
      }

      if (this.player.playerTransform != null)
      {
        Transform _TGunManger = this.player.playerTransform.FindChild("GunManager");                                      
        _TGunManger.SendMessage("FireSimulate", h);     
      }
   }    
~~~~

SetFireSimulate performs the firing events as if a playerLocal would’ve
done them. This forces the remotePlayer to show the same rotation
position and angle that the playerLocal performed in its own client. If
there was a weapon change in the remote player, the change is reflected
here too.

~~~~ {.code}

/// From file: GunManager.cs
    
    function FireSimulate(_hashtable : Hashtable)
    { 
        var weapon : byte = _hashtable["Weapon"];
        var cGun : Gun = guns[weapon].gun;
        
       for(var j : int = 0; j < guns.length; j++)
    {
       guns[j].gun.enabled = false;         }
       cGun.enabled = true;
       currentWeapon = weapon;
       cGun.Local = Local;
       cGun.reloading = _hashtable["reload"];
    
       if (_hashtable["OutPutPoint"] != null)
          cGun.OutputPoint = _hashtable["OutPutPoint"];
                
          cGun.Point = _hashtable["Point"];
          cGun.fireRemote = _hashtable["Fire"];
          cGun.fire = _hashtable["Fire"];
       }
~~~~

The GunManager.cs and Gun.cs scripts both have a boolean variable that
determines how the script will perform (either as a local player or a
remote player). If the Local property is set to false, the script will
act as a remote actor and will not execute anything locally. This is
exemplified in the following lines of code where the fire event is
executed differently depending on whether the player is a local or a
remote.

~~~~ {.code}

/// From file: Gun.cs
    
   function Update()
   {
      if(Local)
      {
         timerToCreateDecal -= Time.deltaTime;
            
       if (Input.GetButtonDown("Fire1") 
              && currentRounds == 0 
              && !reloading && freeToShoot)
          {
         PlayOutOfAmmoSound();
        }
            
        if(Input.GetButtonUp("Fire1"))
        {
                var h : Hashtable;
                h = new Hashtable();
                h.Add(1,Point);
                h.Add(2,fire);
 
                transform.root.SendMessage("Fire",h,  
                                           SendMessageOptions.DontRequireReceiver);
              freeToShoot = true;
           cBurst = burstRate;
        }
            
            HandleReloading();          
            ShotTheTarget();
        }       
        else
        {
                    ///this is the action performed when in remote mode
                ShotTheTarget();
        }
    }
~~~~
