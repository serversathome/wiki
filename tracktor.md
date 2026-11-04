---
title: Tracktor
description: A guide to deploying Tracktor
published: true
date: '2026-11-04T16:23:47.282Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:09:03.806Z'
---

# What is Tracktor?
Tracktor is an open-source web application for comprehensive vehicle management.
Easily track:
⛽ fuel consumption
🛠️ maintenance
🛡️ insurance
📄 regulatory documents for all your vehicles in one place. 

# :logos:docker-icon: 1 · Deploy Tracktor
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
      - CORS_ORIGINS=http://YOUR-SERVER-IP:3333
      - APP_SECRET=CHANGE_ME # generate with: openssl rand -hex 32
```

Replace `YOUR-SERVER-IP` in `CORS_ORIGINS` with your server's IP (or the URL you browse to).

# 2 · Logging In
1. Navigate to `http://IP:3333`
1. On first visit there are no users yet, so a registration form is shown. Enter a username and password and click **Create Account**.
