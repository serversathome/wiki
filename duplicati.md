---
title: Duplicati
description: A guide to deploying Duplicati on TrueNAS as well as via docker compose
published: true
date: 2025-07-28T10:32:45.022Z
tags: 
editor: markdown
dateCreated: 2025-07-28T10:28:55.007Z
---

# ![](/duplicati.png){class="tab-icon"} What is Duplicati?

Dozzle is a small lightweight application with a web based interface to monitor Docker logs. It doesn’t store any log files. It is for live monitoring of your container logs only.

# 1 · Deploy Dozzle
# {.tabset}

## <img src="/truenas.png" class="tab-icon"> TrueNAS
1. 

## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml

services:
  duplicati:
    image: lscr.io/linuxserver/duplicati:latest
    container_name: duplicati
    environment:
      - PUID=0
      - PGID=0
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/duplicati:/config
      - /mnt/tank/backups:/backups
      - /:/source
    ports:
      - 8200:8200
    restart: unless-stopped
```