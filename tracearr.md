---
title: Tracearr
description: A guide to deploying Tracearr
published: true
date: 2025-12-22T13:48:45.009Z
tags: 
editor: markdown
dateCreated: 2025-12-22T13:48:45.008Z
---

# <img src="/tracearr.png" class="tab-icon"> What is Tracearr?

Tracearr is a streaming access manager for Plex, Jellyfin, and Emby that answers one question: *Who's actually using my server, and are they sharing their login?*

Unlike monitoring tools that just show you data, Tracearr is built to detect account abuse. See streams in real-time, flag suspicious activity automatically, and get notified the moment something looks off.

**Session Tracking** — Full history of who watched what, when, from where, on what device. Every stream logged with geolocation data.

**Sharing Detection** — Five rule types catch account sharers:

🚀 Impossible Travel — NYC then London 30 minutes later? That's not one person.
📍 Simultaneous Locations — Same account streaming from two cities at once.
🔀 Device Velocity — Too many unique IPs in a short window signals shared credentials.
📺 Concurrent Streams — Set limits per user. Simple but effective.
🌍 Geo Restrictions — Block streaming from specific countries entirely.

**Real-Time Alerts** — Discord webhooks and custom notifications fire instantly when rules trigger. No waiting for daily reports.

**Stream Map** — Visualize where your streams originate on an interactive world map. Filter by user, server, or time period to zero in on suspicious patterns.

**Trust Scores** — Users earn (or lose) trust based on their behavior. Violations drop scores automatically.

**Multi-Server** — Connect Plex, Jellyfin, and Emby instances to the same dashboard. Manage everything in one place.

**Tautulli Import** — Already using Tautulli? Import your watch history so you don't start from scratch.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Tracearr
```yaml
services:
  tracearr:
    image: ghcr.io/connorgallopo/tracearr:supervised
    shm_size: 256mb
    ports:
      - 3020:3000
    environment:
      - TZ=America/New_York
      - LOG_LEVEL=info
    volumes:
      - /mnt/tank/configs/tracearr/postgres:/data/postgres
      - /mnt/tank/configs/tracearr/redis:/data/redis
      - /mnt/tank/configs/tracearr/data:/data/tracearr
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://127.0.0.1:3000/health"]
      interval: 30s
      timeout: 10s
      start_period: 60s
      retries: 3
```