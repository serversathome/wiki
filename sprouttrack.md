---
title: Sprout Track
description: A guide to deploying Sprout Track
published: true
date: '2026-11-04T23:31:45.828Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:08:46.645Z'
---

# What is Sprout Track?
An application for tracking baby activities, milestones, and development.

# :logos:docker-icon: 1 · Deploy Sprout Track
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
```

> [!NOTE]
> Back up all three folders.

# 2 · First Login
Navigate to `http://IP:3000` to start the setup wizard.

> [!TIP]
> The default security PIN after setup is: 111222

> [!WARNING]
> The admin area at `/family-manager` uses the default password `admin`. Change it right after setup.

::block-steps
1. Enter a **Family Name**
1. Enter a **Family URL**
1. Click **Next**
1. Enter a **PIN**
1. Click **Next**
1. Enter your baby's information
1. Click **Complete Setup**
::


