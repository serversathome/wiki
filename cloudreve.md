---
title: Cloudreve
description: A guide to deploying Cloudreve
published: true
date: '2026-11-04T16:24:01.179Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:03:48.008Z'
---

# What is Cloudreve?

Cloudreve can help you build a self-hosted file management service that is both suitable for private and public use, with support for multiple storage providers and virtual file systems to provide a flexible file management experience.

# :logos:docker-icon: 1 · Deploy Cloudreve
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
    image: postgres:17    
    container_name: cloudreve-postgresql
    restart: unless-stopped
    environment:
      - POSTGRES_USER=cloudreve
      - POSTGRES_DB=cloudreve
      - POSTGRES_HOST_AUTH_METHOD=trust
    volumes:
      - /mnt/tank/configs/cloudreve/db:/var/lib/postgresql/data

  redis:
    image: redis:latest
    container_name: cloudreve-redis
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/cloudreve/redis:/data
```

# 2 · Logging In
1. Navigate to `http://{IP}:5212`
1. Click **Register**


# :logos:youtube-icon: 3 · Video
::block-youtube{url="https://youtu.be/5zWL_cIdmIU"}
::
