---
title: Dispatcharr
description: A guide to deploying Dispatcharr
published: true
date: 2025-10-13T20:44:03.782Z
tags: 
editor: markdown
dateCreated: 2025-10-13T15:42:09.466Z
---

# <img src="/dispatcharr.png" class="tab-icon"> What is Dispatcharr?
Dispatcharr is an open-source powerhouse for managing IPTV streams and EPG data with elegance and control.

Think of Dispatcharr as the *arr family’s IPTV cousin — simple, smart, and designed for streamers who want reliability and flexibility.


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Dispatcharr

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

# 4 · Linking With Your Media Server
1. For Emby, navigate to the **Admin Panel** by clicking the cog wheel in the upper right corner
1. Click **Live TV**
1. Click **+ Add TV Source**
1. Select **M3U**
1. Get the URL from the purple **M3U** button in the **Channels** tab of Dispatcharr
1. Select the options **Import guide directly from the m3u, when available** and **Allow mapping to guide data using channel numbers**
1. Click **Save**

> Each client will need to make sure they have their dashboard set to show Live TV channels
{.is-success}


# <img src="/patreon-light.png" class="tab-icon"> 5 · Video