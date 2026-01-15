---
title: Tracktor
description: A guide to deploying Tracktor
published: true
date: 2026-01-15T15:32:09.986Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:09:03.806Z
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
    image: ghcr.io/javedh-dev/tracktor:latest
    container_name: tracktor-app
    restart: unless-stopped
    ports:
      - "3333:3000"
    volumes:
      - /mnt/tank/configs/tracktor:/data
    environment:
      - TRACKTOR_DEMO_MODE=false
      - FORCE_DATA_SEED=false
      - CORS_ORIGINS=http://10.99.0.242:3333

```

# 2 · Logging In
1. Navigate to `http://IP:3333`
1. The default pin is `123456`