---
title: Trailarr
description: A guide to deploying Trailarr in docker
published: true
date: 2025-07-29T10:31:14.362Z
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
      - "7889:7889"
    restart: unless-stopped
```

# 2 · Trailarr Configuration




# <img src="/youtube.png" class="tab-icon"> 3 · Video