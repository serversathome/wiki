---
title: Cr*n Master
description: A guide to deploying Cr*n Master
published: true
date: '2026-11-04T16:23:17.970Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:04:06.236Z'
---

# What is Cr\*nMaster?
Cronmaster is a tool that helps manage cron jobs, which are scheduled tasks on Unix-like operating systems. It simplifies the process of creating and maintaining these scheduled tasks, often used for automating system maintenance or other repetitive jobs.

# :logos:docker-icon: 1 · Deploy Cr\*nMaster
```yaml
services:
  cronjob-manager:
    image: ghcr.io/fccview/cronmaster:latest
    container_name: cronmaster
    user: "root"
    ports:
      - "40123:3000"
    environment:
      - NODE_ENV=production
      - NEXT_PUBLIC_CLOCK_UPDATE_INTERVAL=30000
      - AUTH_PASSWORD=change-me # Use a strong password, e.g. openssl rand -base64 24
      - HOST_CRONTAB_USER=root
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /mnt/tank/configs/cronmaster/scripts:/app/scripts
      - /mnt/tank/configs/cronmaster/data:/app/data
      - /mnt/tank/configs/cronmaster/snippets:/app/snippets
    pid: "host"
    privileged: true
    restart: unless-stopped
    init: true
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/api/health"]
      interval: 30s
      timeout: 10s
      start_period: 30s
      retries: 3
```

> [!WARNING]
> Cr\*nMaster runs as a privileged root container with the Docker socket, so always set `AUTH_PASSWORD`. Open `http://<IP>:40123` and log in with that password. Host paths for `/app/data` and `/app/scripts` are detected through the Docker socket; if detection fails, set `HOST_DATA_DIR` and `HOST_SCRIPTS_DIR` to the matching host paths.

> [!NOTE]
> Upgrading from 1.x: back up `/mnt/tank/configs/cronmaster` and your host crontab (`crontab -l > crontab.bak`) before pulling the new image. See the [release notes](https://github.com/fccview/cronmaster/releases).
