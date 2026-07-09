---
title: Dispatcharr
description: A guide to deploying Dispatcharr
published: true
date: 2026-07-09T01:30:03.732Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:16.637Z
---

# What is Dispatcharr?

Dispatcharr is an open-source powerhouse for managing IPTV streams, EPG data, and VOD content with elegance and control.

Think of Dispatcharr as the *arr family's IPTV cousin: simple, smart, and designed for streamers who want reliability and flexibility. Beyond consolidating multiple IPTV sources into one interface, it handles Video on Demand with IMDB/TMDB metadata, a plugin system for automation, multi-user access control, and multiple output formats including M3U, XMLTV EPG, Xtream Codes API, and HDHomeRun emulation so media servers like Plex, Emby, and Jellyfin can discover it as a live TV source.

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

> 
> Dispatcharr now auto-generates and persists its own secret key inside the `/data` volume, so you no longer need to set `DJANGO_SECRET_KEY` manually. The compose above runs in all-in-one (AIO) mode with Redis and PostgreSQL bundled inside the single container, which is what most homelabbers want. If you need external Postgres/Redis or TLS-encrypted database connections, there is an optional modular deployment documented in the official Dispatcharr docs.
{.is-info}

# 2 · Logging In

1. Navigate to `http://IP:9191` and set a username and password

# 3 · Adding Channels

1. Click the **+ Add M3U** button
2. Give it a name
3. Use a URL for a m3u list like `https://iptv-org.github.io/iptv/languages/eng.m3u`
4. Set the **Account Type** to `Standard`
5. Click the **Save** button at the bottom
6. Click **Configure Groups** in the message window in the bottom right
7. Click the **Groups** button
8. Click **Save and Refresh**
9. Select all Streams in the Dashboard and click **Create Channels** to turn them into live TV channels

# 4 · Linking With Your Media Server

## 4.1 Emby

1. Navigate to the **Admin Panel** by clicking the cog wheel in the upper right corner
2. Click **Live TV**
3. Click **+ Add TV Source**
4. Select **M3U**
5. Get the URL from the purple **M3U** button in the **Channels** tab of Dispatcharr
6. Select the options **Import guide directly from the m3u, when available** and **Allow mapping to guide data using channel numbers**
7. Click **Save**

> Each client will need to make sure they have their dashboard set to show Live TV channels
{.is-info}


## 4.2 Jellyfin

1. Navigate to the **Admin Panel** by clicking the icon of a man in the upper right corner
2. Click **Dashboard**
3. Click **Live TV**
4. Click the **+** next to **Tuner Devices** at the top
5. Under **Tuner Type** select `M3U Tuner`
6. Get the URL from the purple **M3U** button in the **Channels** tab of Dispatcharr
7. Click **Save**

## 4.3 Plex

Plex does not accept a raw M3U tuner, but Dispatcharr's HDHomeRun emulation lets Plex discover it as a network tuner and record to the Plex DVR.

1. In Plex, go to **Settings** > **Live TV & DVR**
2. Click **Set Up Plex DVR**
3. Plex scans your network for tuners and should detect the Dispatcharr HDHomeRun device automatically. If it does not, choose the option to enter the tuner address manually and paste the URL from the **HDHomeRun** button in the **Channels** tab of Dispatcharr
4. Continue, and when prompted for guide data choose to use an **XMLTV** guide, using the URL from the **EPG** button in Dispatcharr (or map to Plex's own guide data)
5. Finish the setup and Plex will list your Dispatcharr channels under Live TV

> 
> I do not run Plex myself, so if you have this working and can tighten up any of the steps above, please edit this page.
{.is-info}