---
title: Speedtest Tracker
description: A guide to deploy Speed Test Tracker
published: true
date: 2025-09-05T10:41:26.346Z
tags: 
editor: markdown
dateCreated: 2025-09-02T14:27:43.594Z
---

# ![](/speedtest-tracker.png){class="tab-icon"} What is Speedtest Tracker?
Speedtest Tracker is a self-hosted application that monitors the performance and uptime of your internet connection. Build using Laravel and Speedtest CLI from Ookla®, deployable with Docker.

> Generate an app key using this command in the TrueNAS shell: `echo -n 'base64:'; openssl rand -base64 32;`
{.is-info}

# 1 · Deploy Speedtest Tracker
# {.tabset}
## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. 


## <img src="/docker.png" class="tab-icon"> Docker Compose

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
      - APP_KEY=scKR9Sep4myluMpPKvhzJYYcXzRSd0ag
      - DB_CONNECTION=sqlite
    volumes:
      - /mnt/tank/configs/speedtesttracker:/config
```
# 2 · First Login
The default user is `admin@example.com` and the default password is `password`.

> Find additional environment variables see their documentation [here](https://docs.speedtest-tracker.dev/getting-started/environment-variables) 
{.is-info}
