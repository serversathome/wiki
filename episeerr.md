---
title: Episeerr
description: A guide to deploying Episeerr
published: true
date: 2026-01-20T20:28:37.752Z
tags: 
editor: markdown
dateCreated: 2026-01-20T20:27:15.149Z
---

# <img src="/dockpeek.png" class="tab-icon"> What is Episeerr?

Smart episode management for Sonarr - Get episodes as you watch, clean up automatically when storage gets low.

Perfect for:

- Limited storage setups (seedboxes, VPS, budget home servers)
- Preventing runaway downloads that fill your disk
- Automated cleanup based on viewing activity
- Per-season tracking for shows with multiple active seasons
- Granular episode-level control
- Custom rules for different types of shows


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Episeerr
```yaml
services:
  episeerr:
    image: vansmak/episeerr:latest
    environment:
      # Required
      - SONARR_URL=http://your-sonarr:8989
      - SONARR_API_KEY=your_sonarr_api_key
      - TMDB_API_KEY=your_tmdb_api_key
      
      # Optional - For viewing automation
      - TAUTULLI_URL=http://your-tautulli:8181
      - TAUTULLI_API_KEY=your_tautulli_key
      # OR
      - JELLYFIN_URL=http://your-jellyfin:8096
      - JELLYFIN_API_KEY=your_api_key
      - JELLYFIN_USER_ID=your_username
      
      # Optional - For request integration
      - JELLYSEERR_URL=http://your-jellyseerr:5055
      - JELLYSEERR_API_KEY=your_jellyseerr_key
      
      # Optional - Quick links
      - CUSTOMAPP_URL=http://192.168.1.100:8080
      - CUSTOMAPP_NAME=Episeerr
      - CUSTOMAPP_ICON=fas fa-cog

    volumes:
      - /mnt/tank/configs/episeerr/config:/app/config
      - /mnt/tank/configs/episeerr/logs:/app/logs
      - /mnt/tank/configs/episeerr/data:/app/data
      - /mnt/tank/configs/episeerr/temp:/app/temp
    ports:
      - "5002:5002"
    restart: unless-stopped
```