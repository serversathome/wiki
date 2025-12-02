---
title: Technical Difficulties
description: Some common technical issues reported by the community
published: true
date: 2025-12-02T17:37:56.107Z
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

## My qBittorrent won't start
99% of the time this is due to a bad `wg0.conf` file. Make sure to remove any IPv6 information from your wireguard file. See [this section](https://wiki.serversatho.me/qBittorrent#h-2-example-wireguard-wg0conf-file) for an example of a correct wireguard configuration file.

## I get a permissions issue when trying to download a torrent
Make sure your directory structure follows the instructions [here](/Folder-Structure). If it does, click your media folder and apply root:apps 770 permissions recursively. You can also execute this in the shell by running (as root) assuming your pool is named `tank`:
```bash
chown root:apps /mnt/tank/media -R
```

## Unable to add root folder. Folder `/media/movies/` is not writable by user `abc`
This is also a permissions issue. See the solution above.

