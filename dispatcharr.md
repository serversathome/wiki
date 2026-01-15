---
title: Dispatcharr
description: A guide to deploying Dispatcharr
published: true
date: 2026-01-15T15:28:54.723Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:16.637Z
---

# <img src="/dispatcharr.png" class="tab-icon"> What is Dispatcharr?
Dispatcharr is an open-source powerhouse for managing IPTV streams and EPG data with elegance and control.

Think of Dispatcharr as the *arr family’s IPTV cousin — simple, smart, and designed for streamers who want reliability and flexibility.

# 1 · Deploy Dispatcharr
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  dispatcharr:
    image: ghcr.io/dispatcharr/dispatcharr:latest
    container_name: dispatcharr
    restart: unless-stopped
    ports:
      - 9191:9191
    volumes:
      - /mnt/tank/configs/dispatcharr/data:/data
    environment:
      - DJANGO_SECRET_KEY=            #must be 50 charaters and no syntax charaters
      - DISPATCHARR_ENV=aio
      - REDIS_HOST=localhost
      - CELERY_BROKER_URL=redis://localhost:6379/0
      - DISPATCHARR_LOG_LEVEL=info
```

## <img src="/docker.png" class="tab-icon"> Docker + VPN
```yaml
services:
  dispatcharr:
    image: ghcr.io/dispatcharr/dispatcharr:latest
    container_name: dispatcharr
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/dispatcharr/data:/data
    environment:
      - DJANGO_SECRET_KEY=            #must be 50 charaters and no syntax charaters
      - DISPATCHARR_ENV=aio
      - REDIS_HOST=localhost
      - CELERY_BROKER_URL=redis://localhost:6379/0
      - DISPATCHARR_LOG_LEVEL=info
    network_mode: "service:gluetun"

  gluetun:
    image: qmcgaw/gluetun
    container_name: gluetun
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    environment:
      - VPN_SERVICE_PROVIDER=airvpn
      - VPN_TYPE=wireguard
      - WIREGUARD_PRIVATE_KEY=
      - WIREGUARD_PRESHARED_KEY=     #Optional depending on provider/config
      - WIREGUARD_ADDRESSES=
      - SERVER_COUNTRIES=     #Optional depending on provider/config
    ports:
      - 9191:9191
    restart: unless-stopped
```

# 2 · Logging In
1. Navigate to `http://IP:9191` and set a username and password

# 3 · Adding Channels
1. Click the **+ Add M3U** button
1. Give it a name
1. Use a URL for a m3u list like `https://iptv-org.github.io/iptv/languages/eng.m3u`
1. Set the **Account Type** to `Standard`
1. Click the **Save** button at the bottom
1. Click **Configure Groups** in the message window in the bottom right
1. Click the **Groups** button
1. Click **Save and Refresh**
1. Select all Streams in the Dashboard and click **Create Channels** to turn them into live TV channels

# 4 · Linking With Your Media Server
## 4.1 Emby
1. Navigate to the **Admin Panel** by clicking the cog wheel in the upper right corner
1. Click **Live TV**
1. Click **+ Add TV Source**
1. Select **M3U**
1. Get the URL from the purple **M3U** button in the **Channels** tab of Dispatcharr
1. Select the options **Import guide directly from the m3u, when available** and **Allow mapping to guide data using channel numbers**
1. Click **Save**

> Each client will need to make sure they have their dashboard set to show Live TV channels
{.is-success}

## 4.2 Jellyfin
1. Navigate to the **Admin Panel** by clicking the icon of a man in the upper right corner
1. Click **Dashboard**
1. Click **Live TV**
1. Click the **+** next to **Tuner Devices** at the top
1. Under **Tuner Type** select `M3U Tuner`
1. Get the URL from the purple **M3U** button in the **Channels** tab of Dispatcharr
1. Click **Save**

## 4.3 Plex
I do not have Plex, but if any of *you* do and you have these linked please [edit this page](https://github.com/serversathome/wiki/blob/main/dispatcharr.md) and add those instructions here!
