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

<div class="glance">
  <div><span>Port</span><b><code>3000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
</div>

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