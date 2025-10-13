---
title: Dispatcharr
description: A guide to deploying Dispatcharr
published: true
date: 2025-10-13T15:42:09.466Z
tags: 
editor: markdown
dateCreated: 2025-10-13T15:42:09.466Z
---

# <img src="/dispatcharr.png" class="tab-icon"> What is Dispatcharr?


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Dispatcharr

```yaml
services:
  dispatcharr:
    image: ghcr.io/dispatcharr/dispatcharr:latest
    container_name: dispatcharr
    restart: unless-stopped
    ports:
      - 9191:9191
    volumes:
      - /mnt/tank/configs/dispatcharr/data:/data
    environment:
      - DISPATCHARR_ENV=aio
      - REDIS_HOST=localhost
      - CELERY_BROKER_URL=redis://localhost:6379/0
      - DISPATCHARR_LOG_LEVEL=info
```