---
title: Releasarr
description: A guide to deploying Releasarr via docker
published: true
date: 2026-01-15T15:31:23.793Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:08:04.916Z
---

# 🎧 What is Releasarr?
Releasarr is a music release monitoring and management tool designed to help you keep track of your favorite artists and their releases. It integrates multiple music platforms and services to provide a unified view of your music collection and new releases.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Releasarr

```yaml
services:
  Releasarr:
    image: makario1337/releasarr:latest
    container_name: releasarr
    restart: unless-stopped
    ports:
      - 1337:1337
    volumes:
      - /mnt/tank/configs/releasarr/config:/config
      - /mnt/tank/configs/releasarr/logs:/logs
      - /mnt/tank/configs/releasarr/library:/library
    environment:
      APP_PORT: 1337
      APP_WORKERS: 4
```