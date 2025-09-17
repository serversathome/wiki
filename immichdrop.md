---
title: Immich Drop
description: A guide to deploying Immich Drop
published: true
date: 2025-09-17T17:31:52.121Z
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
      IMMICH_ALBUM_NAME: family-dropbox # Optional: auto-add to this album
    env_file:
      - ./.env
    ports:
      - "9002:8080"
    volumes:
      - /mnt/tank/configs/immichdrop:/data
```

## 1.1 Environment Variables

> If you are using Dockge, place these directly underneath the `compose.yaml` section where is says `.env`
{.is-info}


```yaml
HOST=0.0.0.0
PORT=8080
IMMICH_BASE_URL=http://your-immich-server:2283/api
IMMICH_API_KEY=your_immich_api_key_here
MAX_CONCURRENT=3
IMMICH_ALBUM_NAME=family-dropbox # Optional
STATE_DB=/data/state.db
CHUNKED_UPLOADS_ENABLED=true
CHUNK_SIZE_MB=95
```
