---
title: Trailarr
description: A guide to deploying Trailarr in docker
published: true
date: 2025-07-29T10:57:05.267Z
tags: 
editor: markdown
dateCreated: 2025-07-29T10:31:01.593Z
---

# ![](/trailarr.png){class="tab-icon"} What is Trailarr?
Trailarr is a Docker application to download and manage trailers for your Radarr, and Sonarr libraries.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Trailarr

```yaml
services:
  trailarr:
    image: nandyalu/trailarr:latest
    container_name: trailarr
    environment:
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/trailarr:/config
      - /mnt/tank/media/movies:/media/movies
      - /mnt/tank/media/tv:/media/tv
    ports:
      - "7889:7889"
    restart: unless-stopped
```

# 2 · Trailarr Configuration

> Default credentials are **admin:trailarr**
{.is-info}

## 2.1 Adding Connections
1. Navigate to **Connections** and add your Radarr/Sonarr URL and API Key
1. Click **Test**
1. Click the folder icon in the **Path Mappings** seciton and use the default provided path
1. Click **Test** again
1. Click **Submit**


# <img src="/patreon-light.png" class="tab-icon"> 3 · Video