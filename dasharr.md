---
title: Dasharr
description: A guide to deploying Dasharr in docker
published: true
date: 2025-08-04T11:22:31.095Z
tags: 
editor: markdown
dateCreated: 2025-08-04T11:22:20.098Z
---

# ![](/dasharr.png){class="tab-icon"} What is Dasharr?
Dasharr is a unified media and network dashboard that brings all your media management services and network monitoring together in one place. Following the *arr naming convention (like Radarr, Sonarr), Dasharr provides a beautiful, responsive interface to monitor and manage your entire media stack and network infrastructure.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Dasharr


```yaml
services:
  dasharr:
    image: schenanigans/dasharr:latest
    container_name: dasharr
    ports:
      - "3000:3000"
    volumes:
      - /mnt/tank/configs/dasharr:/app/config
    restart: unless-stopped
```