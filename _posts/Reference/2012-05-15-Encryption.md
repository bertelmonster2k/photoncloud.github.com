---
layout: article
title: Encryption
categories: [photon-server, photon-cloud, references]
tags: [setup, how-to, installation, sample]
---
{% include globals %}

Encryption
----------

Photon is able to encrypt messages between client and server - a
must-have security-measure to send authentication input and other
sensible user-data.

As encryption causes some overhead and is not always needed, you chose
to enable and use it per operation.

### Enabling Encryption

To use encryption, client and server will have to exchange their public
keys to calculate a shared secret. This is necessary per individual
connection and simple to achieve.

When you know that you client needs encryption at some time, you best
establish it when it connects. OnStatusChanged() is a good place for
this:

~~~~ {.code}
    
    public void OnStatusChanged(StatusCode returnCode)
    {
        // handle returnCodes for connect, disconnect and errors (non-operations)
        switch (returnCode)
        {
            case StatusCode.Connect:
                this.DebugReturn("Connect(ed)");
                this.peer.EstablishEncryption();
                // ...
    
~~~~

The library takes care of sending and handling the required keys. When
this finishes, the client library will call OnStatusChanged with either
of these codes:

~~~~ {.code}
    
    public void OnStatusChanged(StatusCode returnCode)
    {
        // handle returnCodes for connect, disconnect and errors (non-operations)
        switch (returnCode)
        {
            // ... 

            case StatusCode.EncryptionEstablished:
                // encryption is now available
                // this is used per operation call
                break;
            case StatusCode.EncryptionFailedToEstablish:
                // if this happens, get in contact with Exit Games
                break;
    
~~~~

### Calling Encrypted Operations

Once encryption is established, it can be used on a per-operation basis.

Lite, as the "default" implementation, does not require sensible
user-data, so you won't find operations in that API supporting
encryption. However, you can call any operation by OpCustom():

~~~~ {.code}
    
    // OpCustom has an overload with encryption parameter:
    this.peer.OpCustom(myOpCode, opParameters, sendReliable, channel, encrypt);
    
~~~~

In the application frameworks we provide (Lite, Loadbalancing, etc), the
server automatically responds encrypted, if a operation was sent
encrypted. This makes it safe to fetch critical data by simply
requesting with encryption turned on.

We usually don't encrypt events. In our default logic, these go to all
players of a room, so the content can't be too secret to send openly.
