---
title: Palmr
description: A guide to deploying Palmr on docker
published: true
date: 2025-07-07T17:53:06.736Z
tags: 
editor: markdown
dateCreated: 2025-07-07T17:51:37.781Z
---

![palmr.png](/palmr.png)

# What is Palmr?
Palmr is a flexible and open-source alternative to file transfer services like WeTransfer, SendGB, Send Anywhere, and Files.fm.

# Installation
```yaml
services:
  palmr:
    image: kyantech/palmr:latest
    container_name: palmr
    environment:
      - ENABLE_S3=false
      - ENCRYPTION_KEY=changemechangemechangemechangeme
      - PALMR_UID=568
      - PALMR_GID=568
    ports:
      - 5487:5487 # Web port
    volumes:
      - /mnt/tank/configs/palmr:/app/server
    restart: unless-stopped
```
> If the logs say `"Waiting for API..."` give it 5 minutes to wait before stopping the container
{.is-info}
