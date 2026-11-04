---
title: Speedtest Tracker
description: A guide to deploy Speed Test Tracker
published: true
date: '2026-11-04T16:17:58.119Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:08:44.488Z'
---

# What is Speedtest Tracker?
Speedtest Tracker is a self-hosted application that monitors the performance and uptime of your internet connection. Build using Laravel and Speedtest CLI from Ookla®, deployable with Docker.

# 1 · Deploy Speedtest Tracker
:::block-tabs
::block-tab{label="TrueNAS" icon="simple-icons:truenas"}
1. Generate an app key by running `openssl rand -hex 16` in the TrueNAS shell
1. Change the **Config Storage** to **Host Path**

::

::block-tab{label="Docker Compose" icon="simple-icons:docker"}
```yaml
services:
  speedtest-tracker:
    image: lscr.io/linuxserver/speedtest-tracker:latest
    restart: unless-stopped
    container_name: speedtest-tracker
    ports:
      - 8080:80
    environment:
      - PUID=568
      - PGID=568
      - APP_KEY= # Generate with: openssl rand -hex 16
      - APP_URL=http://your-server-ip:8080
      - DB_CONNECTION=sqlite
      - SPEEDTEST_SCHEDULE=0 */6 * * *
    volumes:
      - /mnt/tank/configs/speedtesttracker:/config
```

::

:::

# 2 · First Login
The default user is `admin@example.com` and the default password is `password`. Change both right after your first login.

> [!NOTE]
> Find additional environment variables in their [documentation](https://docs.speedtest-tracker.dev/getting-started/environment-variables) 

