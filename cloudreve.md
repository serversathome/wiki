---
title: Cloudreve
description: A guide to deploying Cloudreve
published: true
date: 2025-12-01T12:43:49.416Z
tags: 
editor: markdown
dateCreated: 2025-12-01T12:42:04.280Z
---

# <img src="/cloudreve.png" class="tab-icon"> What is Cloudreve?

Cloudreve can help you build a self-hosted file management service that is both suitable for private and public use, with support for multiple storage providers and virtual file systems to provide a flexible file management experience.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Cloudreve
```yaml
services:
  cloudreve:
    image: cloudreve/cloudreve:latest
    container_name: cloudreve-backend
    depends_on:
      - postgresql
      - redis
    restart: unless-stopped
    ports:
      - 5212:5212
      - 6888:6888
      - 6888:6888/udp
    environment:
      - CR_CONF_Database.Type=postgres
      - CR_CONF_Database.Host=postgresql
      - CR_CONF_Database.User=cloudreve
      - CR_CONF_Database.Name=cloudreve
      - CR_CONF_Database.Port=5432
      - CR_CONF_Redis.Server=redis:6379
    volumes:
      - /mnt/tank/configs/cloudreve/data:/cloudreve/data

  postgresql:
    # Best practice: Pin to major version. 
    # NOTE: For major version jumps:
    # backup & consult https://www.postgresql.org/docs/current/pgupgrade.html 
    image: postgres:17    
    container_name: postgresql
    restart: unless-stopped
    environment:
      - POSTGRES_USER=cloudreve
      - POSTGRES_DB=cloudreve
      - POSTGRES_HOST_AUTH_METHOD=trust
    volumes:
      - /mnt/tank/configs/cloudreve/db:/var/lib/postgresql/data

  redis:
    image: redis:latest
    container_name: redis
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/cloudreve/redis:/data
```

# 2 · Logging In
1. Navigate to `http://{IP}:5212`
1. Click **Register**


# <img src="/youtube.png" class="tab-icon"> 3 · Video 
