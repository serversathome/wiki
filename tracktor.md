---
title: Tracktor
description: A guide to deploying Tracktor
published: true
date: 2026-10-28T12:00:00.000Z
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

<div class="glance">
  <div><span>Port</span><b><code>3333</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/javedh-dev/tracktor"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

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
      - CORS_ORIGINS=http://YOUR-SERVER-IP:3333
      - APP_SECRET=CHANGE_ME # generate with: openssl rand -hex 32
```

Replace `YOUR-SERVER-IP` in `CORS_ORIGINS` with your server's IP (or the URL you browse to).

# 2 · Logging In
1. Navigate to `http://IP:3333`
1. On first visit there are no users yet, so a registration form is shown. Enter a username and password and click **Create Account**.