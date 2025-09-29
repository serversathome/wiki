---
title: CrossWatch
description: A guide to deploying CrossWatch
published: true
date: 2025-09-29T17:07:58.918Z
tags: 
editor: markdown
dateCreated: 2025-09-29T17:06:20.606Z
---

# <img src="/crosswatch.png" class="tab-icon"> What is CrossWatch?
Synchronize your data across Plex, Jellyfin, SIMKL, Trakt, and more. Keep your movies and shows in sync, no matter where you watch. 

# <img src="/docker.png" class="tab-icon"> 1 · Deploy CrossWatch

```yaml
services:
  crosswatch:
    image: ghcr.io/cenodude/crosswatch:latest
    container_name: crosswatch
    ports:
      - 8787:8787
    environment:
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/crosswatch:/config
    restart: unless-stopped