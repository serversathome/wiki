---
title: Cleanuparr
description: A guide to deploying Cleanuparr via docker
published: true
date: 2025-06-28T13:00:14.162Z
tags: 
editor: markdown
dateCreated: 2025-06-28T12:56:06.603Z
---

![cleanuparr.png](/cleanuparr.png)

# What is Cleanuparr?

Automated Download Management. Automatically clean up unwanted, stalled, and malicious downloads from your \*arr applications and download clients. Keep your queues clean and your media library safe.

# Installation

```yaml
services:
  cleanuparr:
    image: ghcr.io/cleanuparr/cleanuparr:latest
    container_name: cleanuparr
    restart: unless-stopped
    ports:
      - 11011:11011
    volumes:
      - /mnt/tank/configs/cleanuparr:/config
      - /mnt/tank/media:/media
    environment:
      - PORT=11011
      - BASE_PATH=
      - PUID=568
      - PGID=568
      - UMASK=022
      - TZ=America/New_York
```