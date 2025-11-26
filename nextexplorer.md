---
title: nextExplorer
description: A guide to deploying nextExplorer
published: true
date: 2025-11-26T11:20:18.417Z
tags: 
editor: markdown
dateCreated: 2025-11-03T14:41:55.290Z
---

# ![](/nextexplorer.png){class="tab-icon"} What is nextExplorer?

A modern, self-hosted file explorer with secure access control, polished UX, and a Docker-friendly deployment story.
# <img src="/docker.png" class="tab-icon"> 1 · Deploy nextExplorer
```yaml
services:
  nextexplorer:
    image: nxzai/explorer:latest
    container_name: nextexplorer
    restart: unless-stopped
    ports:
      - 3000:3000
    environment:
      - NODE_ENV=production
      # Optional: match the container user/group to your host IDs
      # - PUID=1000
      # - PGID=1000
    volumes:
      - /mnt/tank/configs/nextexplorer:/cache
      - /mnt/tank:/mnt/Tank
```