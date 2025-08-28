---
title: Tracktor
description: A guide to deploying Tracktor
published: true
date: 2025-08-28T18:34:56.491Z
tags: 
editor: markdown
dateCreated: 2025-08-28T18:34:56.490Z
---

# <img src="/tracktor.png" class="tab-icon"> What is Tracktor?
Tracktor is an open-source web application for comprehensive vehicle management.
Easily track:
⛽ fuel consumption
🛠️ maintenance
🛡️ insurance
📄 regulatory documents for all your vehicles in one place. 

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Tracktor
```yaml
services:
  app:
    image: ghcr.io/javedh-dev/tracktor:dev
    container_name: tracktor-app
    restart: unless-stopped
    ports:
      - 3333:3000
    volumes:
      - /mnt/tank/configs/tracktor:/config
    environment:
      - PUBLIC_API_BASE_URL=http://[enter-your-ip]:3333
```