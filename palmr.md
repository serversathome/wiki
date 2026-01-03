---
title: Palmr
description: A guide to deploying Palmr on docker
published: true
date: 2026-01-03T01:42:42.939Z
tags: 
editor: markdown
dateCreated: 2025-07-07T17:51:37.781Z
---

> **DO NOT USE AS CONTAINER HAS BEEN DEPRECATED DUE TO THE CHANGES IN MINIO**
{.is-danger}


# <img src="/palmr.png" class="tab-icon"> What is Palmr?
Palmr is a flexible and open-source alternative to file transfer services like WeTransfer, SendGB, Send Anywhere, and Files.fm.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Palmr
```yaml
services:
  palmr:
    image: kyantech/palmr:latest
    container_name: palmr
    environment:
      - ENABLE_S3=false
      - ENCRYPTION_KEY=changemechangemechangemechangeme
      - PUID=568
      - PGID=568
    ports:
      - 5487:5487 # Web port
    volumes:
      - /mnt/tank/configs/palmr:/app/server
    restart: unless-stopped
```
> If the logs say `1/2 - Palmr. starting, be patient...` wait 5 minutes or so before stopping the container
{.is-info}
