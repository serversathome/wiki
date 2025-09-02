---
title: Speedtest Tracker
description: A guide to deploy Speed Test Tracker
published: true
date: 2025-09-02T14:29:41.085Z
tags: 
editor: markdown
dateCreated: 2025-09-02T14:27:43.594Z
---

# ![](/speedtest-tracker.png){class="tab-icon"} What is Speedtest Tracker?
Speedtest Tracker is a self-hosted application that monitors the performance and uptime of your internet connection. Build using Laravel and Speedtest CLI from Ookla®, deployable with Docker.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Speedtest Tracker

```yaml
services:
  speedtest-tracker:
    image: lscr.io/linuxserver/speedtest-tracker:latest
    restart: unless-stopped
    container_name: speedtest-tracker
    ports:
      - 8080:80
      - 8443:443
    environment:
      - PUID=568
      - PGID=568
      - APP_KEY=1tZoe4IX020scKR9Sep4myluMpPKvhzJYYcXzRSd0ag=
      - DB_CONNECTION=sqlite
    volumes:
      - /mnt/tank/configs/speedtesttracker:/config
```