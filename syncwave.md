---
title: Syncwave
description: A guide to deploying Syncwave
published: true
date: 2026-01-15T15:32:01.146Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:08:51.089Z
---

# <img src="/syncwave.png" class="tab-icon"> What is Syncwave?

Syncwave is a real-time kanban board that's simple and beautiful. 

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Syncwave
```yaml
services:
  syncwave:
    image: syncwave/syncwave
    container_name: syncwave
    restart: unless-stopped
    ports:
      - 8080:8080
    volumes:
      - /mnt/tank/configs/syncwave-data:/data

```