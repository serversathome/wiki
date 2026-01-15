---
title: Tugtainer
description: A guide to deploying Tugtainer
published: true
date: 2026-01-15T15:32:13.516Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:09:08.384Z
---

# What is Tugtainer?
An application for automating docker container updates with a web UI.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Tugtainer

```yaml
services:
  tugtainer:
    container_name: tugtainer
    image: quenary/tugtainer:latest
    restart: unless-stopped
    ports:
      - '9412:80'
    volumes:
    	- /var/run/docker.sock:/var/run/docker.sock
      - /mnt/tank/configs/tugtainer:/tugtainer
```

