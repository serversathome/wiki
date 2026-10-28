---
title: Sprout Track
description: A guide to deploying Sprout Track
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:08:46.645Z
---

# <img src="/sprout-track.png" class="tab-icon"> What is Sprout Track?
An application for tracking baby activities, milestones, and development.

<div class="glance">
  <div><span>Port</span><b><code>3000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/Oak-and-Sprout/sprout-track"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Sprout Track
```yaml
services:
  sprout-track:
    image: sprouttrack/sprout-track:latest
    container_name: sprout-track
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - /mnt/tank/configs/sprouttrack/db:/db
      - /mnt/tank/configs/sprouttrack/env:/app/env
      - /mnt/tank/configs/sprouttrack/files:/app/Files
    environment:
      - NODE_ENV=production
      - DATABASE_PROVIDER=sqlite
      - ENABLE_NOTIFICATIONS=true
      - APP_URL=http://your-server-ip:3000
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

> The `env` folder holds the app's generated `.env` file and encryption keys. Back it up together with `db`. Optional push-notification settings (`VAPID_*`, `ROOT_DOMAIN`) and PostgreSQL support are covered in the [upstream Docker docs](https://github.com/Oak-and-Sprout/sprout-track/blob/main/documentation/Admin-Documentation/docker-deployment.md).
{.is-info}

# 2 · First Login
Navigate to `http://IP:3000` to start the setup wizard.

> The default security PIN after setup is: 111222
{.is-success}

> The admin area at `/family-manager` uses the default password `admin`. Change it right after setup.
{.is-warning}

1. Enter a **Family Name**
1. Enter a **Family URL**
1. Click **Next**
1. Enter a **PIN**
1. Click **Next**
1. Enter your baby's information
1. Click **Complete Setup**

