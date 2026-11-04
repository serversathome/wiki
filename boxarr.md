---
title: Boxarr
description: A guide to deploying Boxarr
published: true
date: '2026-11-04T23:31:40.593Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:03:34.981Z'
---

# What is Boxarr?

Boxarr monitors weekly box office charts and seamlessly integrates with Radarr to ensure your media library always has what people want to watch. No more manual searching for popular movies - Boxarr handles it automatically.

# :logos:docker-icon: 1 · Deploy Boxarr
```yaml
services:
  boxarr:
    image: ghcr.io/iongpt/boxarr:latest
    container_name: boxarr
    ports:
      - 8898:8888
    volumes:
      - /mnt/tank/configs/boxarr:/config
    restart: unless-stopped
    environment:
      - TZ=America/New_York
```

# 2 · Boxarr Configuration

::block-steps
1. Enter Radarr URL as `http://IP:port`
1. Enter Radarr **API Key**
1. Select Radarr **Root Folder**
1. Select **Default Quality Profile**
1. Select number of movies per week
1. Click **Save Configuration**
::

# 3 · Find Movies

Since this is the first time the container will run, click the **Weeks** tab and select **Update Last Week**. If you do not run this manually the container will update your movies on the next scheduled run.

# ![](/_files/patreon-light.png =28x) 4 · Video
[![](/2025-09-11-boxarr-automatically-add-weekl-promo-card.png)](https://www.patreon.com/posts/boxarr-add-box-138680972)
