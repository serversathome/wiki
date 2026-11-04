---
title: Emby Wrapped
description: A guide to deploying Emby Wrapped
published: true
date: '2026-11-04T16:24:16.589Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:04:33.448Z'
---

# What is Emby Wrapped?

A beautiful, Spotify Wrapped-style year-in-review experience for your Emby media server. See your viewing stats, top shows, favorite genres, and more in an interactive, animated presentation.

> [!NOTE]
> **Prerequisite:** Emby Wrapped reads its stats from the **Playback Reporting** plugin. In Emby go to **Dashboard → Plugins → Catalog**, install **Playback Reporting** and restart Emby. Then create an API key under **Dashboard → API Keys**. The plugin only tracks plays from the moment it is installed.

# :logos:docker-icon: 1 · Deploy
```yaml
services:
  emby-wrapped:
    image: ghcr.io/davidtorcivia/emby-wrapped-ftp:latest
    container_name: emby-wrapped
    ports:
      - "3000:3000"
    environment:
      - EMBY_URL=http://your-emby-server:8096
      - EMBY_API_KEY=your-api-key-here
      - TMDB_API_KEY=  # Optional: for enhanced poster images
      - PUBLIC_URL=    # Optional: public URL for share links
    volumes:
      - /mnt/tank/configs/emby-wrapped/music:/app/static/music:ro  # Optional: MP3s for custom background music
    restart: unless-stopped
```
