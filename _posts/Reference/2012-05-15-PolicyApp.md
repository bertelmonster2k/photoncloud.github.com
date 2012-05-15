---
layout: article
title: Policy Application
categories: [photon-server, references]
tags: [setup, how-to, installation]
---
{% include globals %}

Policy Application
------------------

The Policy Application runs on Photon to send the crossdomain.xml.
Webplayer platforms like Unity Webplayer, Flash and Silverlight request
authorization before they contact a server.

The actual file sent in response to a policy request is loaded from:
deploy\\Policy\\Policy.Application\\assets. There is a special file for
Silverlight and one for Unity and Flash.

### Configuration

The default setup from the SDK will start the Policy Application. The
assemblies of which are in the deploy\\Policy folder and the
configuration to load it is:

~~~~ {.code}
    
        <Applications>
            <!-- [other Application nodes] -->

            <!-- Flash & Silverlight Policy Server -->
            <Application
                Name="Policy"
                BaseDirectory="Policy\Policy.Application"
                Assembly="Policy.Application"
                Type="Exitgames.Realtime.Policy.Application.Policy">
            </Application>
        </Applications>
    
~~~~

### Ports

Policy requests are usually done behind the scenes on TCP Port 843 and
943 (Silverlight), so these two have to be open as well. This includes
Windows Security settings, other Firewalls in software and hardware. If
you host Photon in a Cloud, check the Security settings of that as well.
Amazon's EC2 has Security Groups to restrict access to ports.
