---
title: nextExplorer
description: A guide to deploying nextExplorer
published: true
date: 2026-01-15T15:30:22.986Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:06:42.032Z
---

# ![](/nextexplorer.png){class="tab-icon"} What is nextExplorer?

A modern, self-hosted file explorer with secure access control, polished UX, and a Docker-friendly deployment story.

<div class="glance">
  <div><span>Port</span><b><code>3000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy nextExplorer
```yaml
services:
  nextexplorer:
    image: nxzai/explorer:latest
    container_name: nextexplorer
    restart: unless-stopped
    ports:
      - 3000:3000
    environment:
      - NODE_ENV=production
      - PUID=568
      - PGID=568
    volumes:
    	- /mnt/tank/configs/nextexplorer/config:/config
      - ./cache:/cache
      - /mnt/tank:/mnt/tank
```