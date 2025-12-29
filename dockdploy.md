---
title: Dock-Dploy
description: A guide to deploying Dock-Dploy
published: true
date: 2025-12-29T15:51:21.396Z
tags: 
editor: markdown
dateCreated: 2025-12-29T15:51:21.396Z
---

# <img src="/dockpeek.png" class="tab-icon"> What is Dock-Dploy?
A web-based tool for building, managing, and converting Docker Compose files, configuration files, and schedulers. 
# <img src="/docker.png" class="tab-icon"> 1 · Deploy Dock-Dploy
```yaml
services:
  dock-dploy:
    image: hhftechnology/dock-dploy:latest
    container_name: dock-dploy
    restart: unless-stopped
    ports:
      - 3000:3000
    environment:
      - NODE_ENV=production
```