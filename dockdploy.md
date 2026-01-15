---
title: Dock-Dploy
description: A guide to deploying Dock-Dploy
published: true
date: 2026-01-15T15:28:56.278Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:18.698Z
---

# What is Dock-Dploy?
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