---
title: Boxarr
description: A guide to deploying Boxarr
published: true
date: 2025-09-11T17:51:18.833Z
tags: 
editor: markdown
dateCreated: 2025-09-11T14:02:01.158Z
---

# ![](/boxarr.png){class="tab-icon"} What is Boxarr?

Boxarr monitors weekly box office charts and seamlessly integrates with Radarr to ensure your media library always has what people want to watch. No more manual searching for popular movies - Boxarr handles it automatically.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Boxarr
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
1. Enter Radarr URL as `http://IP:port`
1. Enter Radarr **API Key**
1. Select Radarr **Root Folder**
1. Select **Default Quality Profile**
1. Select number of movies per week
1. Click **Save Configuration**

# 3 · Find Movies

Since this is the first time the container will run, click the **Weeks** tab and select 