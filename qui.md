---
title: Qui
description: A guide to deploying Qui
published: true
date: 2025-09-22T15:28:55.036Z
tags: 
editor: markdown
dateCreated: 2025-09-22T15:28:55.036Z
---

# <img src="/autobrr.png" class="tab-icon"> What is Qui?
A fast, modern web interface for qBittorrent. Supports managing multiple qBittorrent instances from a single, lightweight application.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Qui
```yaml
services:
  qui:
    image: ghcr.io/autobrr/qui:latest
    container_name: qui
    restart: unless-stopped
    ports:
      - "7476:7476"
    volumes:
      - /mnt/tank/configs/qui:/config
```

# 2 · Logging In


# <img src="/patreon-light.png" class="tab-icon"> 3 · Video