---
title: Syncwave
description: A guide to deploying Syncwave
published: true
date: 2025-09-30T16:01:01.577Z
tags: 
editor: markdown
dateCreated: 2025-09-30T16:01:01.577Z
---

# What is Syncwave?

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