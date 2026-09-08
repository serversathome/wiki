---
title: Speedtest Tracker
description: A guide to deploy Speed Test Tracker
published: true
date: 2026-01-15T15:31:56.091Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:08:44.488Z
---

# ![](/speedtest-tracker.png){class="tab-icon"} What is Speedtest Tracker?
Speedtest Tracker is a self-hosted application that monitors the performance and uptime of your internet connection. Build using Laravel and Speedtest CLI from Ookla®, deployable with Docker.

<div class="glance">
  <div><span>Port</span><b><code>8080</code></b></div>
  <div><span>Deploy via</span><b>TrueNAS app or Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/linuxserver/docker-speedtest-tracker"><i class="mdi mdi-github"></i>Project</a>
    <a href="https://docs.speedtest-tracker.dev/getting-started/environment-variables"><i class="mdi mdi-book-open-variant"></i>Docs</a>
  </div>
</div>


# 1 · Deploy Speedtest Tracker
# {.tabset}
## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Generate an app key by running `openssl rand -hex 16` in the TrueNAS shell or use the example key from the Docker Compose tab
1. Change the **Config Storage** to **Host Path**


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

> Find additional environment variables in their [documentation](https://docs.speedtest-tracker.dev/getting-started/environment-variables) 
{.is-info}
