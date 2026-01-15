---
title: Qui
description: A guide to deploying Qui
published: true
date: 2026-01-15T15:31:11.669Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:46.966Z
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
1. Navigate to `http://{IP}:7476`
1. Set a username and password
1. Add a new instance


# <img src="/patreon-light.png" class="tab-icon"> 3 · Video
[![](/2025-09-29-qui-a-better-qbit-interface-promo-card.png)](https://www.patreon.com/posts/qui-better-qbit-139484651)