---
layout: article
title: Boot Camp Demo: Frags And Respawns
categories: [photon-cloud, tutorials]
tags: [sample, how-to, unity, demo]
---
{% include globals %}


When a PlayerLocal shoots at a remote player, the information is
received by SoldierDamageControl in the graphical representation of the
playerRemote. This sends a message

~~~~ {.code}

    
    _usePhoton.SendMessage("DownLifeGranade", ActorNr);

    
~~~~

OR

~~~~ {.code}


    _usePhoton.SendMessage("DownLife",ActorNr);

    
~~~~

with the ID of the playerLocal that has the same ID. The playerLocal
that has the same ID receives the information and decreases his/her
health depending on the weapon dealing damage.

If a player shoots at another player, the remotePlayer that's shot will
send an event to notify everyone else that he’s been damaged. All
players receive this and the player that's hit (same actorNumber)
reduces health because of the event notification.

~~~~ {.code}

/// From file: SoldierDamageControl.js

///RELEVANT CODE IS HIGHLIGHTED
    
class SoldierDamageControl extends MonoBehaviour
{
    public var life : float;
    public var Respawn : Vector3[];
    public var Parent : GameObject;
    public var hitTexture : Texture2D;
    public var blackTexture : Texture2D;
    public var Skin : GUISkin;
    public var Local : boolean;
    
    public var Soldier : GameObject;
    public var Gun : GameObject;
    
    public var KilledStyle : GUIStyle;
    public var ActorNr : int;
    public var ActorName : String;
    public var DisplayKillYou : boolean;
    
    private var hitAlpha : float;
    private var blackAlpha : float;
    private var AtackerName : String;
    private var actorText : GameObject;
    private var actorTextOffset : Vector3 = new Vector3(0,1.0f,0);
    
    private var recoverTime : float;
    
    public var hitSounds : AudioClip[];
    public var dyingSound : AudioClip;
    
    function SetActorNr(DataActor : Hashtable)
    {
        ActorNr = DataActor["actornr"];
        ActorName = DataActor["actorname"];
        if(actorText == null && gameObject.layer != 13)
        {
            this.actorText = Instantiate(Resources.Load("ActorName"));
          this.actorText.name = ActorNr+"";
            actorText.transform.localScale = new Vector3(1.4 / 8f, 1.4 / 8f, 1.4 / 8f);
           }
        
    }
    
       function Start()
    {
        SoldierController.dead = false;
        hitAlpha = 0.0;
        blackAlpha = 0.0;
        life = 1.0;
        KilledStyle = Skin.GetStyle("Killed");
    }
    
    function HitSoldierGranade(hit : String)
    {

        if(GameManager.receiveDamage)
        {
                   if(!audio.isPlaying)
            {
               if(life < 0.5 && (Random.Range(0, 100) < 30))
               {
                   audio.clip = dyingSound;
               }
               else
               {
                audio.clip = hitSounds[Random.Range(0, hitSounds.length)];
               }
                
               audio.Play();
               }

              if(gameObject.layer != 13)
              {          
                     var _usePhoton : GameObject;
                     _usePhoton = GameObject.Find("_GameManager/UsePhoton");                
                     _usePhoton.SendMessage("DownLifeGranade", ActorNr);
                     
                     return;
              }
         
            life -= 1.0;

            recoverTime = (1.0 - life) * 10.0;
            
            if(hit == "Dummy")
            {
                TrainingStatistics.dummiesHit++;
            }
            else if(hit == "Turret")
            {
                TrainingStatistics.turretsHit++;
            }
            
            if(life <= 0.0)
            {
                AtackerName = hit;
                SoldierController.dead = true;
                DisplayKillYou = true;
                
            }
            
        }
    }   
    
       function HitSoldier(hit : String)
    {
        if(GameManager.receiveDamage)
        {              
                    if(!audio.isPlaying)
            {
               if(life < 0.5 && (Random.Range(0, 100) < 30))
               {
                  audio.clip = dyingSound;
               }
               else
               {
                  audio.clip = hitSounds[Random.Range(0, hitSounds.length)];
               }
                
                  audio.Play();
            }
        
              if(gameObject.layer != 13)
              {  
                      var _usePhoton : GameObject;
                      _usePhoton = GameObject.Find("_GameManager/UsePhoton");                
                      _usePhoton.SendMessage("DownLife",ActorNr);
                
                return;
              }

            life -= 0.05;
            
            recoverTime = (1.0 - life) * 10.0;
            
            if(hit == "Dummy")
            {
               TrainingStatistics.dummiesHit++;
            }
            else if(hit == "Turret")
            {
               TrainingStatistics.turretsHit++;
            }
            
            if(life <= 0.0)
            {
                AtackerName = hit;
                DisplayKillYou = true;
                SoldierController.dead = true;  
            }
            
        }
    }
    
    function DisplayCountDown(time : int)
    {
        yield WaitForSeconds (time);
           DisplayKillYou = false;
    }
    
    function Update()
    {   
        if(actorText != null)
        {
        
            var textMesh : TextMesh =  actorText.GetComponent(TextMesh);
            textMesh.text = ActorName;
            /// text looking into oposite direction of camera
            this.actorText.transform.position = transform.position + this.actorTextOffset;
            var camDiff : Vector3 = actorText.transform.position - Camera.main.transform.position;
            var lookAt : Vector3 = actorText.transform.position + camDiff;
            this.actorText.transform.LookAt(lookAt);
        }
       
        recoverTime -= Time.deltaTime;
        
        if(recoverTime <= 0.0)
        {
            life += life * Time.deltaTime;
            
            life = Mathf.Clamp(life, 0.0, 1.0);
            
            hitAlpha = 0.0;
        }
        else
        {
            hitAlpha = recoverTime / ((1.0 - life) * 10.0);
        }
    
        if(!SoldierController.dead)
        { 
            blackAlpha = 0.0;
 
            return;
        }
        
        
        blackAlpha += Time.deltaTime;
        
        if(blackAlpha >= 1.0)
        {
            life = 1.0;
            hitAlpha = 0.0;
            blackAlpha = 0.0;
            recoverTime = 0;
            if(SoldierController.dead)
            {
                  var _DeadUsePhoton : GameObject;
                  _DeadUsePhoton = GameObject.Find("_GameManager/UsePhoton");
               _DeadUsePhoton.SendMessage("Dead", Respawn[Random.Range(0, 13)]);      
            }   
            SoldierController.dead = false;
     }      

        
        if(DisplayKillYou)
        {
            DisplayCountDown(7);
        }
    }
    
    function OnGUI()
    {
        if(DisplayKillYou)
        {
            if(AtackerName != "Player"){
                GUI.Label(Rect (Screen.width - 200,60,200,50),AtackerName+" killed you",KilledStyle);
            }else
            {
                GUI.Label(Rect (Screen.width - 200,60,200,50),"killed yourself",KilledStyle);
            }
        }
        if(!GameManager.receiveDamage) return;
        if(gameObject.layer != 13){ 
            return;
        }

        var oldColor : Color;
        var auxColor : Color;
        oldColor = auxColor = GUI.color;
        
        auxColor.a = hitAlpha;
        GUI.color = auxColor;
        GUI.DrawTexture(new Rect(0, 0, Screen.width, Screen.height), hitTexture);
        
        auxColor.a = blackAlpha;
        GUI.color = auxColor;
        GUI.DrawTexture(new Rect(0, 0, Screen.width, Screen.height), blackTexture);
        

        
        GUI.color = oldColor;
    }   
}

~~~~

~~~~ {.code}

/// From file: usePhoton.cs
    
///Here, SoldierDamageControl executes an OpRaiseEvent  to send a message notifying dealt damage to a specified target and depending on which weapon is selected.

    void DownLife(int actorNr)
    {
        Hashtable evInfo = new Hashtable();
       evInfo["T"] =  actorNr;
    evInfo["H"] =(byte) 0;        
       GameInstance.peer.OpRaiseEvent(Constants.EV_HIT, evInfo, true);      
    }

    
    void DownLifeGranade(int actorNr)
    {
        Hashtable evInfo = new Hashtable();
      evInfo["T"] =  actorNr;
        evInfo["H"] = (byte)1;        
        GameInstance.peer.OpRaiseEvent(Constants.EV_HIT, evInfo, true);     
    }
~~~~

The damage message is received by Game.cs in the EV\_HIT case. That
particular block of code sends a message to the playerLocal that matches
the target’s id sent in the message.

~~~~ {.code}

/// From file: Game.cs
    
    case Constants.EV_HIT:
      
         Hashtable data = (Hashtable)photonEvent[(byte)LiteEventKey.Data];
         int target = (int)data["T"];
         ///Note that the sent target id and the playerLocal Id are compared to see if the dealt damage is meant for this player
         if (target == LocalPlayer.playerID) 
         {
             switch ((byte)data["H"])
             {
                 case 0:
                   PlayerLocal.DownLifeHitSoldier(p.playerRemote.Name);
                   break;

                 case 1:
                    PlayerLocal.DownLifeHitSoldierGranade(p.playerRemote.Name);
                    break;
 
              }
          }
      
          break;
~~~~

When playerLocal’s Health has reached 0, SoldierDamageControl sends a
message to usePhoton calling the Dead function and supplying it with the
new respawn position for the player. An event of type EV\_FRAG is
broadcast so playerRemotes with the same id are removed.

~~~~ {.code}

/// From file: usePhoton.cs
    
    void Dead(Vector3 spawnPosition)
    {
       GameInstance.PlayerLocal.transform.position = spawnPosition;
       GameInstance.PlayerLocal.MoveOp(Constants.EV_FRAG, true);        
    }
~~~~

This event reaches Game.cs and enters the EV\_FRAG block of code in
EventAction and executes SetSpawnPosition.

~~~~ {.code}

/// From file: Game.cs
    
case Constants.EV_FRAG:         
   p.playerRemote.Dead = true;     
   p.playerRemote.SetSpawnPosition((Hashtable)photonEvent[(byte)LiteEventKey.Data]);
   break;
~~~~

SetSpawnPosition instantiates a new representation of the dead
playerRemote and moves the playerRemote to its new spawn position. The
representation is removed after 8 seconds have elapsed and the Dead flag
for the remote player is reset to false.

~~~~ {.code}

/// From file: PlayerRemote.cs
    
     internal void SetSpawnPosition(Hashtable evData)
     {
    transform.eulerAngles = new Vector3(90, transform.rotation.y, 0);
    _DeadSplash = (GameObject)Instantiate(DeadSpalsh, transform.position + new Vector3(0, 1f, 0), transform.rotation);
       Destroy(_DeadSplash, 8);

       Vector3 p = GetPosition((float[])evData[Constants.STATUS_PLAYER_POS]);
       Quaternion r = GetRotation((float[])evData[Constants.STATUS_PLAYER_ROT]);
    this.pos = p;   
       this.transform.position = p; 
       this.rot = r;                
       this.realPos = p;
       this.lastUpdateTime = UnityEngine.Time.time;

    targetpos = GetPosition((float[])evData[Constants.STATUS_TARGET_POS]);
       SendMessage("SetTargetPos", targetpos);            

       transform.position = pos;
       transform.localRotation = rot;
         Dead = false;      
      }
~~~~
