---
title: Immich Drop
description: A guide to deploying Immich Drop
published: true
date: 2025-09-30T14:48:35.455Z
tags: 
editor: markdown
dateCreated: 2025-09-01T11:48:30.786Z
---

# What is Immich Drop?

Immich Drop is a tiny web app that acts as an anonymous upload gateway to your Immich server. You spin it up alongside your Immich instance, and it provides a clean, minimal interface where anyone can drag-and-drop photos and videos. These files are uploaded directly into your Immich library, preserving original dates and handling duplicates intelligently.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Immich Drop

```yaml
services:
  immich-drop:
    image: ghcr.io/nasogaa/immich-drop:latest
    container_name: immich-drop
    restart: unless-stopped
    environment:
      - HOST=0.0.0.0
      - PORT=8080
      - IMMICH_BASE_URL=http://your-immich-server:2283/api
      - IMMICH_API_KEY=
      - MAX_CONCURRENT=3
      - STATE_DB=/data/state.db
      - CHUNKED_UPLOADS_ENABLED=true
      - CHUNK_SIZE_MB=95

    ports:
      - "9002:8080"
    volumes:
      - /mnt/tank/configs/immichdrop:/data
```
1. For the `IMMICH_BASE_URL` use the private IP address of Immich if it is reachable from the Immich Drop container (otherwise use the public URL)
1. To create the Immich API key, navigate to Immich → User Icon (top right) → Account Settings → API Key