---
layout: article
title: Boot Camp Demo - Scripts
categories: [photon-cloud, tutorials]
tags: [sample, how-to, unity, demo]
---
{% include globals %}

### Enums.cs

This script contains an enumeration KeyState that contains all possible
animation states depending on the player’s current keyboard input. If
the player runs, we switch the state to Running; if the player walks, we
switch the state to Walking, so on and so forth. We only notify when the
state changes, trying to reduce the message traffic amongst peers. If
there’s no state change, the present state remains until told otherwise.

~~~~ {.code}

    /// from file: Enums.cs
    
    public enum KeyState : byte
    {
        Still = 0x0,            ///00000000  0
        Runing = 0x1,           ///00000001  1
        WalkBack = 0x2,         ///00000010  2
        RuningBack = 0x3,       ///00000011  3
        WalkStrafeLeft = 0x5,   ///00000101  5
        WalkStrafeRight = 0x6,  ///00000110  6
        RunStrafeLeft = 0x10,   ///00001010  10
        RunStrafeRight = 0x11,  ///00001011  11
        Left = 0x4,             ///00000100  4
        Right = 0x8,            ///00001000  8
        Jump = 0x10,            ///00010000  16
        Free1 = 0x20,           ///00100000  32
        Free2 = 0x40,           ///01000000  64
        NoChange = 0x80,        ///10000000  128
        Vertical = 0x3,         ///00000011  3
        Horizontal = 0xC,       ///00001100  12
        Walking = 0xF,          ///00001111  15
    }
~~~~

### ActorAnimator.cs

This script plays the animations corresponding to the current KeyState
the remote player (script playerRemote.cs) has.

~~~~ {.code}

/// from file: ActorAnimator.cs
    
void Update()
{
   if (playerRemote.keyState == Enums.KeyState.Walking) { WalkFoward(); }
   if (playerRemote.keyState == Enums.KeyState.WalkBack) { WalkBack(); }
   if (playerRemote.keyState == Enums.KeyState.Runing) { Run(); }
   if (playerRemote.keyState == Enums.KeyState.RuningBack) { RunBack(); }
}
    
public void WalkFoward()
{
   if (crouch)
   {
      if (aim)
      { 
       animation["CrouchWalkAim"].speed = 1.0F;
       animation.CrossFade("CrouchWalkAim");
      }
      else
      {
         animation["CrouchWalk"].speed = 1.0F;
         animation.CrossFade("CrouchWalk");
      }
   }
   else 
   {
      if (aim)
      {
         animation["WalkAim"].speed = 1.0F;
         animation.CrossFade("WalkAim");
      }
      else
      {
       animation["Walk"].speed = 1.0F;
       animation.CrossFade("Walk");
      }
   }
}
~~~~

To perform the actual synchronization, the following scripts were
created both for the local and remote player:

Player.cs: This script handles players connected to a specific room. It
also contains a reference variable (public PlayerRemote playerRemote;)
that points to PlayerRemote.cs. Messages sent to remote players are
accessed via this particular reference.

PlayerLocal.cs: This script is responsible for sending all fire,
position and rotation events as well as animation KeyStates such as
Crouch or Aim. The events are routed to their remote counter-part using
the player ID and calling PlayerRemote.cs to sync each remote player’s
state.

PlayerRemote.cs: This script receives the event and property change
messages that are sent by its local counter-part (PlayerLocal.cs).


### Game.cs

Handling of incoming messages and their proper assignment to each player
is done by Game.cs. The Game class encapsulates the usage of the
PhotonPeer, event handling and the game’s base logic. To make it work
properly, it must be implemented in a game loop that regularly calls
Game.Update().

~~~~ {.code}

/// From file: Game.cs
    
/// Update must be called by a gameloop (a single thread), so it can handle automatic movement and networking.
public void Update()
{
    this.Service();
}

    
/// From file: usePhoton.cs
    
/// GameLoop
void Update()
{
    if (GameInstance == null)
    {
        return;
    }
        
    /// Game Update call
    GameInstance.Update();
    UpdatePositions();
}
~~~~

### PlayerLocal.cs

As an example, Player’s position must be regularly updated, and such
action is performed from PlayerLocal.cs. Player names are set only when
someone enters the room. States like Crouch or Aim are updated when
their state changes and are event driven. PlayerLocal.cs is also
responsible for notifying such updates to the other payers.

~~~~ {.code}

/// From file: PlayerLocal.cs
    
    /// Updates position and rotation by using the Move() function and the keystate by getting the result for the ReadKeyboardInput() 
    ///function. Messages are sent using Anim() only after checking if the current state is different from the previous one.                
     public void Update () 
     {
        try
        {
           if (this.engine != null)
           {
              this.ReadKeyboardInput();
              this.nameText.text = this.engine.LocalPlayer.name;
              this.Move();
                 this.Anim();
           }
        }
        catch (Exception e)
        {
           Debug.Log(e);
        }
     }
   
    /// Reads keyboard actions to change the keyState state and verify if the chat window is open or not void ReadKeyboardInput()
    {
         if (ChatScreen.chat) 
         {          
           keyState = KeyState.Still;
            return;
        }
        
        if (Input.GetAxis("Horizontal") > 0.1f)
        {
            if (Walk)
            {
                keyState = KeyState.WalkStrafeRight;
            }
            else
            {
                keyState = KeyState.RunStrafeRight;
            }
        }
        else if (Input.GetAxis("Horizontal") < -0.1f)
        {
            if (Walk)
            {
                keyState = KeyState.WalkStrafeLeft;
            }
            else
            {
                keyState = KeyState.RunStrafeLeft;
            }
        }
        else if (Input.GetAxis("Vertical") > 0.1)
        {
            if (Walk)
            {
                keyState = KeyState.Walking;
            }
            else
            {
                keyState = KeyState.Runing;
            }
        }
        else if (Input.GetAxis("Vertical") < -0.1)
        {
            if (Walk)
            {
                keyState = KeyState.WalkBack;
            }
            else
            {
                keyState = KeyState.RuningBack;
            }
        }
        else
            keyState = KeyState.Still;
    }

    /// Sends the EV_MOVE message each time that Time.time is greater than nextMoveTime
    /// nextMoveTime is 0.1f greater than Time.time’s current value.
    private void Move()
    {
       if (Time.time > this.nextMoveTime)
       {
        this.MoveOp(Constants.EV_MOVE, false);            
       }
    }
    
    public void MoveOp(byte operationCode, bool reliable)
    {
        
        float[] Position = GetPosition(this.transform.position);
        float[] Rotation = GetRotation(this.transform.rotation);
        float[] TargetPosition = GetPosition(this.SoldierTarget.position);
            
        Hashtable evInfo = new Hashtable();

        evInfo.Add(Constants.STATUS_PLAYER_POS, (float[])Position);
        evInfo.Add(Constants.STATUS_PLAYER_ROT, (float[])Rotation);
        evInfo.Add(Constants.STATUS_TARGET_POS, (float[])TargetPosition);        
                
        engine.peer.OpRaiseEvent(operationCode, evInfo, reliable);
            
        this.nextMoveTime = Time.time + 0.1f;
    }

    private void Anim()
    {
    if (keyState != lastKeyState)
    {          
           /// This message is only sent if the current state and the current one are different.
           Hashtable evInfo = new Hashtable();
           evInfo.Add(Constants.STATUS_PLAYER_KEYSTATE, (byte)keyState);
           engine.peer.OpRaiseEvent(Constants.EV_ANIM, evInfo, true);
           lastKeyState = keyState;
       }
    }
~~~~

The following lines describe the path followed by a message to be
correctly received by the playerRemote.cs on another machine:

~~~~ {.code}

<code Message Path>

/// We’ll describe the Aim event. First, the event that triggers the message sending routine is captured in the local player.

///SoldierController.js runs the Getuserinputs function. We used sendmessage because the original routine was programmed that way in the original game. This message is received by PlayerLocal.cs in its SetAim function with aim’s current value.
    function GetUserInputs()
    {
        ///Check if the user is aiming the weapon
    aim = Input.GetButton("Fire2") && !dead;
    if(oldaim != aim)
    {
        SendMessage("SetAim",aim, SendMessageOptions.DontRequireReceiver);
        oldaim = aim;
    }
    }
    
/// PlayerLocal.cs executes SetAim and the message is sent to the peer via OpSetPropertiesOfActor
public void SetAim(bool _Flag)
{     
    Hashtable evInfo = new Hashtable();
    evInfo.Add("A", (bool)_Flag);
    engine.peer.OpSetPropertiesOfActor(this.engine.LocalPlayer.playerID, evInfo, 
                                       true, 0);
}

/// LitePeer.cs gathers the received data in the properties hashtable and sends a SetProperties operation (For more details on SetProperties see Photon Lite documentation)
public virtual short OpSetPropertiesOfActor(int actorNr, Hashtable properties, bool broadcast, byte channelId)
{
      Hashtable wrap = new Hashtable();
      wrap.Add(LiteOpKey.Properties, properties);
      wrap.Add(LiteOpKey.ActorNr, actorNr);
      if (broadcast)
      {
          wrap.Add(LiteOpKey.Broadcast, broadcast);
      }

      return OpCustom((byte)LiteOpCode.SetProperties, wrap, true, channelId);
}

/// The LiteEventCode SetProperties is received by EventAction and sent to playerRemote.cs with the ID corresponding to the player that originated the message.
public void EventAction(byte eventCode, Hashtable photonEvent)
{
   if (eventCode != Constants.EV_MOVE)
   {
      this.DebugReturn("EventAction() " + eventCode);
   }

            int actorNr = (int)photonEvent[(byte)LiteEventKey.ActorNr];
            
            /// get the player that raised this event
            Player p;
            this.Players.TryGetValue(actorNr, out p);

            switch (eventCode)
            {
                
                case (byte)LiteEventCode.SetProperties:
                    Hashtable properties =         
                            (Hashtable)photonEvent[(byte)LiteEventKey.Properties];
            p.playerRemote.SetProperties(properties);                   
                    break;
            }
        }
/// SetProperties assigns the corresponding value which in turn is used by the script to represent the action in actoranimator.cs
internal void SetProperties(Hashtable properties)
{
   if (properties.ContainsKey("N"))
      this.Name = (string)properties["N"];
        
   if (properties.ContainsKey("W"))           
      this.currentWeapon = (byte)properties["W"];
        
   if (properties.ContainsKey("C"))           
      this.CrouchSatus = (bool)properties["C"];        
        
   if (properties.ContainsKey("R"))           
    this.InAir = (bool)properties["R"];        
        
   if (properties.ContainsKey("A"))           
    this.Aim = (bool)properties["A"];  
}

/// In the following example, If the player is in an Idle state and the message sends an aim state change, the StandingAim animation is executed
    public void Idle()
    {
        if (animation.IsPlaying("Walk"))
            animation.Blend("Walk", 0.0F, 0.1F);

        if (animation.IsPlaying("Run"))
            animation.Blend("Run", 0.0F, 0.1F);

        if (animation.IsPlaying("StandingJump"))
            animation.Blend("StandingJump", 0.0F, 0.1F);

        if (aim)
        {
            animation["StandingAim"].speed = 1.0F;
            animation.CrossFade("StandingAim");
        }
        else
        {
            animation.CrossFade("Standing");
        }
    }
</code  Message Path>
~~~~
