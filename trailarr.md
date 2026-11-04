---
title: Trailarr
description: A guide to deploying Trailarr in docker
published: true
date: '2026-11-04T23:32:22.204Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:09:05.969Z'
---

# What is Trailarr?
Trailarr is a Docker application to download and manage trailers for your Radarr, and Sonarr libraries.

# :logos:docker-icon: 1 · Deploy Trailarr

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

> [!NOTE]
> Default credentials are **admin:trailarr**

## 2.1 Adding Connections

::block-steps
1. Navigate to **Connections** and add your Radarr/Sonarr URL and API Key
1. Change the **Monitor Type** to `Missing`
1. Click **Test**
1. Click the folder icon in the **Path Mappings** seciton and use the default provided path
1. Click **Test** again
1. Click **Submit**
::


# ![](/_files/patreon-light.png =28x) 3 · Video

[![](/2025-07-29-trailarr-automatically-manage-m-promo-card.png)](https://www.patreon.com/posts/trailarr-manage-135217577)
