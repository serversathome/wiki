---
title: Dash
description: A guide to deploying Dash
published: true
date: 2026-01-26T14:16:20.520Z
tags: 
editor: markdown
dateCreated: 2026-01-26T14:16:20.520Z
---

# <img src="/dashdot.png" class="tab-icon"> What is Dash?

A simple, modern server dashboard, primarily used by smaller private servers.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Dash
```yaml
services:
  dash:
    image: mauricenino/dashdot:latest
    restart: unless-stopped
    container_name: dash
    privileged: true
    ports:
      - '3001:3001'
    volumes:
      - /mnt/tank/configs/dash:/mnt/host:ro
```