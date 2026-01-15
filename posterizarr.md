---
title: Posterizarr
description: A guide to deploying Posterizarr
published: true
date: 2026-01-15T15:30:56.593Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:26.738Z
---

# ![](/posterizarr.png){class="tab-icon"} What is Posterizarr?

Automate the creation of beautiful, textless posters for your Plex, Jellyfin, or Emby library.

Posterizarr is a PowerShell script with a full Web UI that automates generating images for your media library. It fetches artwork from Fanart.tv, TMDB, TVDB, Plex, and IMDb, focusing on textless images and applying your own custom overlays and text.

- User-Friendly Web UI: Manage settings, monitor activity, and trigger runs from a browser.
- Multiple Media Servers: Supports Plex, Jellyfin, and Emby.
- Kometa Integration: Organizes assets in a Kometa-compatible folder structure.
- Smart Integration: Trigger runs from Tautulli, Sonarr, and Radarr.


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

# <img src="/patreon-light.png" class="tab-icon"> 2 · Video

[![2025-11-17-edit-poster-art-with-posterizarr-promo-card.png](/2025-11-17-edit-poster-art-with-posterizarr-promo-card.png)](https://www.patreon.com/posts/edit-poster-art-143777654)