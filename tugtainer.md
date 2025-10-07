---
title: Tugtainer
description: A guide to deploying Tugtainer
published: true
date: 2025-10-07T13:48:19.095Z
tags: 
editor: markdown
dateCreated: 2025-10-07T13:48:19.095Z
---

# What is Tugtainer?
An application for automating docker containers updates with a web UI.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Tugtainer

```yaml
services:
  app:
    container_name: tugtainer
    image: quenary/tugtainer:latest
    restart: unless-stopped
    ports:
      - '9412:80'
    volumes:
    	- /var/run/docker.sock:/var/run/docker.sock
      - /mnt/tank/configs/tugtainer:/tugtainer
```

