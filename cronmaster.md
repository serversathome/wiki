---
title: Cr*n Master
description: A guide to deploying Cr*n Master
published: true
date: 2025-08-28T18:16:04.141Z
tags: 
editor: markdown
dateCreated: 2025-08-28T18:15:53.169Z
---

# <img src="/cronmaster.png" class="tab-icon"> What is Cr\*nMaster?
Cronmaster is a tool that helps manage cron jobs, which are scheduled tasks on Unix-like operating systems. It simplifies the process of creating and maintaining these scheduled tasks, often used for automating system maintenance or other repetitive jobs.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Cr\*nMaster
```yaml
services:
  cronjob-manager:
    image: ghcr.io/fccview/cronmaster:1.3.0
    container_name: cronmaster
    user: "root"
    ports:
      - "40123:3000"
    environment:
      - NODE_ENV=production
      - DOCKER=true
      - HOST_PROJECT_DIR=/mnt/tank/configs/cronmaster
      - NEXT_PUBLIC_CLOCK_UPDATE_INTERVAL=30000
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /mnt/tank/configs/cronmaster/scripts:/app/scripts
      - /mnt/tank/configs/cronmaster/data:/app/data
      - /mnt/tank/configs/cronmaster/snippets:/app/snippets
    pid: "host"
    privileged: true
    restart: unless-stopped
    init: true
```