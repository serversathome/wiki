---
title: Cinephage
description: A guide to deploying Cinephage
published: true
date: 2025-12-29T22:07:42.497Z
tags: 
editor: markdown
dateCreated: 2025-12-29T22:07:42.497Z
---

# <img src="/cinephage.png" class="tab-icon"> What is Cinephage?

The AIO solution to your self hosted media gathering needs 
# <img src="/docker.png" class="tab-icon"> 1 · Deploy Cinephage
```yaml
services:
  cinephage:
    image: ghcr.io/moldytaint/cinephage:latest
    container_name: cinephage
    restart: unless-stopped
    user: '568:568'
    security_opt:
      - no-new-privileges:true
    ports:
      - '3000:3000'
    volumes:
      - /mnt/tank/configs/cinephage/data:/app/data
      - /mnt/tank/configs/cinephage/logs:/app/logs
      - /mnt/tank/media:/media
    environment:
      - ORIGIN=http://10.99.0.242:3000
      - TZ=America/New_York
```