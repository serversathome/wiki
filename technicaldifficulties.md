---
title: Technical Difficulties
description: Some common technical issues reported by the community
published: true
date: 2025-12-02T17:18:45.756Z
tags: 
editor: markdown
dateCreated: 2025-12-02T17:10:47.700Z
---


## I can't reach the qBittorrent WebUI
qBit needs to know where you are coming from to grant you access. That variable is found on the `- VPN_LAN_NETWORK=` line in the docker compose file. The entry needs to be in CIDR notation. In other words, if your TrueNAS server is at 192.168.1.25 your entry should be `192.168.1.0/24`.

If you are VPN-ing back into your home net you will not be on the same network as your qBit instance. You will need to add the LAN network you are coming in on to the `- VPN_LAN_NETWORK=` line in a comma-separated list. 

For example if you are using tailscale, you need to add `100.64.0.0/10` to your `- VPN_LAN_NETWORK=` line. Wireguard is typically `10.8.0.0/16`. 

## I get `Unauthorized` when visiting the qBittorrent web UI
This generally a good thing! Try clicking on the URL in the browser bar and hit <kbd>ENTER</kbd> and the interface should load.

##