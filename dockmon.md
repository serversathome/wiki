---
title: Dockmon
description: A guide to deploying Dockmon
published: true
date: 2026-01-15T15:29:00.135Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:23.225Z
---

# ![](/dockmon.png){class="tab-icon"} What is Dockmon?

A comprehensive Docker container monitoring and management platform with real-time monitoring, intelligent auto-restart, multi-channel alerting, and complete event logging.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Dockmon
```yaml
services:
  dockmon:
    image: darthnorse/dockmon:latest
    container_name: dockmon
    restart: unless-stopped
    ports:
      - "8001:443"
    environment:
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/dockmon:/app/data
      - /mnt/tank/stacks:/stacks
      - /var/run/docker.sock:/var/run/docker.sock
    healthcheck:
      test: ["CMD", "curl", "-k", "-f", "https://localhost:443/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```


# 2 · Logging In
1. Navigate to `https://{IP}:8001`
1. The default user is `admin` and the default password is `dockmon123`