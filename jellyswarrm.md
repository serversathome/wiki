---
title: Jellyswarrm
description: A guide to deploy Jellyswarrm
published: true
date: 2025-12-01T21:52:11.992Z
tags: 
editor: markdown
dateCreated: 2025-12-01T21:52:11.992Z
---

# <img src="/jellyswarrm.png" class="tab-icon"> What is Jellyswarrm?
Jellyswarrm is a reverse proxy that lets you combine multiple Jellyfin servers into one place. If you’ve got libraries spread across different locations or just want everything together, Jellyswarrm makes it easy to access all your media from a single interface.
# <img src="/docker.png" class="tab-icon"> 1 · Deploy Jellyswarrm
```yaml
services:
  jellyswarrm:
    image: ghcr.io/llukas22/jellyswarrm:latest
    container_name: jellyswarrm
    restart: unless-stopped
    ports:
      - 3000:3000
    volumes:
      - /mnt/tank/configs/jellyswarrm:/app/data
    environment:
      - JELLYSWARRM_USERNAME=admin
      - JELLYSWARRM_PASSWORD=jellyswarrm # ⚠️ Change this in production!
```