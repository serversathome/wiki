---
title: Sortarr
description: A guide to deploying Sortarr
published: true
date: 2026-01-17T15:46:01.206Z
tags: 
editor: markdown
dateCreated: 2026-01-17T15:46:01.206Z
---

# What is Sortarr?
Sortarr is a lightweight web dashboard for Sonarr and Radarr that helps you understand how your media library uses storage. It is not a Plex tool, but it is useful in Plex setups for spotting oversized series or movies and comparing quality vs. size trade-offs.



# <img src="/docker.png" class="tab-icon"> 1 · Deploy Sortarr
```yaml
services:
  sortarr:
    image: ghcr.io/jaredharper1/sortarr:latest
    container_name: sortarr
    ports:
      - 9595:8787
    environment:
      - ENV_FILE_PATH=/data/Sortarr.env
      - CACHE_SECONDS=300
    volumes:
      - /mnt/tank/configs/sortarr:/data
    restart: unless-stopped
```