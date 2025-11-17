---
title: Posterizarr
description: A guide to deploying Posterizarr
published: true
date: 2025-11-17T13:46:25.136Z
tags: 
editor: markdown
dateCreated: 2025-11-17T13:46:25.136Z
---

# ![](/posterizarr.png){class="tab-icon"} What is Posterizarr?


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Posterizarr
```yaml
services:
  posterizarr:
    hostname: posterizarr
    container_name: posterizarr
    environment:
      - TZ=America/New_York
      - TERM=xterm
      - RUN_TIME=disabled
    image: ghcr.io/fscorrupt/posterizarr:latest
    restart: unless-stopped
    user: 568:568
    ports:
      - 8000:8000
    volumes:
      - /mnt/tank/configs/posterizarr:/config:rw
      - /mnt/tank/configs/posterizarr/assets:/assets:rw
      - /mnt/tank/configs/posterizarr/assetsbackup:/assetsbackup:rw
      - /mnt/tank/configs/posterizarr/manualassets:/manualassets:rw
```