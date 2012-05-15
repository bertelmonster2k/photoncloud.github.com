---
layout: article
title: Firewall Settings
categories: [photon-server, references]
tags: [setup, how-to, installation]
---
{% include globals %}

Windows Firewall
----------------

If Photon should be accessible from remote machines (escpecially through
the internet), make sure the Windows Firewall does not block
connections.

To check settings, open the "Windows Firewall" in the Windows Control
Panel. Click "Allow a program through Windows Firewall" and verify
PhotonSocketServer is listed and checked.

![](../img/Firewall-Settings.jpg) Windows 2008 Firewall Settings

Cloud Security Settings
-----------------------

If Photon should run within a Cloud Service, you have to make sure the
UDP and TCP ports are available.

In Amazon EC2, the Security Group should at least open these ports:

![](../img/Amazon-Ec2-Security-Settings.jpg) Amazon EC2 Settings for
Photon (use View Image to see full)
