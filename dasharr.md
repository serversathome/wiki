---
title: Dasharr
description: A guide to deploying Dasharr in docker
published: true
date: 2025-08-13T14:58:05.708Z
tags: 
editor: markdown
dateCreated: 2025-08-04T11:22:20.098Z
---

# ![](/dasharr.png){class="tab-icon"} What is Dasharr?
Dasharr is a unified media and network dashboard that brings all your media management services and network monitoring together in one place. Following the *arr naming convention (like Radarr, Sonarr), Dasharr provides a beautiful, responsive interface to monitor and manage your entire media stack and network infrastructure.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Dasharr


```yaml
services:
  dasharr:
    image: schenanigans/dasharr:latest
    container_name: dasharr
    ports:
      - "3000:3000"
    volumes:
      - /mnt/tank/configs/dasharr:/app/config
    restart: unless-stopped
```

# 2 · Configure Dasharr

1. Navigate to http://IP:3000
1. Click **Configuration** in the navigation
1. Select a service to configure
1. Enter your service URL and API key/token
1. Click **Test Connection** to verify
1. Click **Save** Configuration

# <img src="/patreon-light.png" class="tab-icon"> 3 · Video

[![](/2025-08-13-dasharr-the-ultimate-dashboard--promo-card.png)](https://www.patreon.com/posts/dasharr-ultimate-136235243)