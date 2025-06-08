---
title: AirVPN
description: Instructions on how to use AirVPN to create a wireguard config for torrenting
published: true
date: 2025-06-08T18:39:20.520Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:38:29.347Z
---

![](/sky_1010x227.png)
# What is AirVPN?
An OpenVPN and WireGuard based VPN operated by activists in defense of net neutrality, privacy and against censorship.

This impenetrable tunnel prevents criminal organizations, your ISP or even your government from spying on your communications.


> Want to say thanks? Sign up using [my link](https://airvpn.org/?referred_by=669318) and I get free VPN time!
{.is-success}

# Creating a .conf File

To use AirVPN, sign up and generate a Wireguard Configuration from the **Client Area** tab, then the **Config Generator** box (_not_ the Eddie client).

![](https://wiki.hydrology.cc/airvpnconf.png)

1. Click the **Advanced** slider
1. Change the **IP Exit Layer** to **IPv4 Only**
1. Switch the **Wireguard** slide
1. Scroll down the the single server section and select one which is fastest (usually the ones which are 20000 MB/s)
1. Scroll all the way to the bottom and click **Generate**
1. Once it saves the file, rename it **wg0.conf.**

> You will need to copy these contents to the folder housing your qBittorrent config
{.is-warning}


# Port Forwarding

The point of using AirVPN is the port forward for qBittorrent. Go the **Client Area** tab, then the **Ports** box, and click **Manage**.

![](/screenshot_from_2024-02-23_09-42-58.png)

1. At the bottom, click **Request a new port**
1. Change the **Device** name to the device which you just created the wireguard config file for
1. Change the **IP Layer** to **IPv4 Only**
1. After you add this to your TrueNAS and qBit configuration, come back and click **Test open**. It should look like this if its working:

![](https://wiki.hydrology.cc/portforward.png)


