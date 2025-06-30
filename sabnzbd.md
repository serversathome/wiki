---
title: SABnzbd
description: A guide to deploying SABnzbd via TrueNAS or docker
published: true
date: 2025-06-30T22:30:23.228Z
tags: 
editor: markdown
dateCreated: 2025-06-30T22:21:23.261Z
---

![2025-06-30-sabnzbd-logo](https://github.com/user-attachments/assets/ccfbcf99-65eb-4eed-85dc-447423728b22)

# What is SABnzbd?
SABnzbd is a free, open-source Usenet download manager. It automates the process of downloading, verifying, repairing, extracting, and organizing files from Usenet based on NZB files. SABnzbd runs as a web-based application and integrates with automation tools like **Sonarr**, **Radarr**, and **Lidarr** for seamless media management.

# Prerequisites
You need:

- The media dataset created according to the [Folder-Structure](/Folder-Structure) guide
- Two subdirectories in the `downloads` directory called `complete` and `incomplete`

You can create the two sub-directories with this command (as root in the TrueNAS Shell):

```bash
mkdir -p /mnt/tank/media/downloads/{complete,incomplete}
```

Once the subdirectories have been created, give them the proper permissions by navigating to the **Datasets** tab in TrueNAS and editing the permissions for the `media` dataset and applying them recursively.

# Installation
# {.tabset}
## TrueNAS
![Screenshot 2025-06-30 222117](https://github.com/user-attachments/assets/be440efb-f8cc-4bac-af3c-de0f592bc932)

1. Change the **SABnzbd Config Storage** to **Host Path**
1. Add an **Additional Storage** host path pointed at your media dataset

## Docker
```yaml
services:
  sabnzbd:
    image: lscr.io/linuxserver/sabnzbd:latest
    container_name: sabnzbd
    environment:
      - PUID=568
      - PGID=568
      - TZ=Europe/Amsterdam
    volumes:
      - /mnt/tank/configs/sabnzbd:/config
      - /mnt/tank/media/:/media
    ports:
      - 8080:8080
    restart: unless-stopped
```
## Hotio + VPN
```yaml
services:
  sabnzbd:
    container_name: sabnzbd
    image: ghcr.io/hotio/sabnzbd
    ports:
      - 8080:8080
    environment:
      - PUID=568
      - PGID=568
      - UMASK=002
      - TZ=Europe/Amsterdam
      - WEBUI_PORTS=8080/tcp,8080/udp
      - VPN_ENABLED=true #
      - VPN_CONF=wg0 #
      - VPN_PROVIDER=generic #
      - VPN_LAN_NETWORK=192.168.1.0/24 #
      - VPN_LAN_LEAK_ENABLED=false #
      - VPN_EXPOSE_PORTS_ON_LAN #
      - VPN_AUTO_PORT_FORWARD=false #
      - VPN_AUTO_PORT_FORWARD_TO_PORTS= #
      - VPN_FIREWALL_TYPE=auto #
      - VPN_HEALTHCHECK_ENABLED=false #
      - VPN_NAMESERVERS= #
      - PRIVOXY_ENABLED=false #
      - UNBOUND_ENABLED=false #
      - UNBOUND_NAMESERVERS #
    cap_add:
      - NET_ADMIN
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1 #
      - net.ipv6.conf.all.disable_ipv6=1 #
    volumes:
      - /mnt/tank/configs/sabnzbd:/config
      - /mnt/tank/media/:/media
```
> When you start this container it will fail until you add the VPN config file. See the Example Wireguard wg0.conf File section below
{.is-warning}

### Example Wireguard wg0.conf File

This is an example of how your `wg0.conf` file should look like. If there's a lot of extra stuff, remove it unless you know what it's there for.

```yaml
[Interface]
Address = 10.171.142.169
PrivateKey = supersecretkey
MTU = 1320
DNS = 10.128.0.1

[Peer]
PublicKey = supersecretkey
PresharedKey = supersecretkey
Endpoint = 1.2.3.4:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 15
```

> For more clarification on these values, [look here](https://hotio.dev/containers/sabnzbd/#__tabbed_2_1)
{.is-info}

> If you are using the hotio container for qBit with a VPN this file needs to be added to the `wireguard` folder (in the folder holding the sabnzbd configuration files) before the container can run
{.is-warning}

# Setting up SABnzbd
1. Navigate to `http://(truenas ip):port`
1. Select your preferred language
1. Enter your provider details and test the connection
1. Click the gear icon to edit the folder settings
1. Set the completed downloads to `/media/downloads/complete`
1. Set the temporary downloads to `/media/downloads/incomplete`
1. Save the changes
