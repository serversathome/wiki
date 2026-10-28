---
title: Speedtest Tracker
description: A guide to deploy Speed Test Tracker
published: true
date: 2026-10-28T12:00:00.000Z
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
    <a href="https://github.com/alexjustesen/speedtest-tracker"><i class="mdi mdi-github"></i>Project</a>
    <a href="https://docs.speedtest-tracker.dev/getting-started/environment-variables"><i class="mdi mdi-book-open-variant"></i>Docs</a>
  </div>
</div>


# 1 · Deploy Speedtest Tracker
# {.tabset}
## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Generate an app key by running `echo "base64:$(openssl rand -base64 32)"` in the TrueNAS shell and paste the whole output (including `base64:`) into the app key field
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
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
      - APP_KEY= # Generate with: echo "base64:$(openssl rand -base64 32)"
      - APP_URL=http://your-server-ip:8080
      - DB_CONNECTION=sqlite
      - SPEEDTEST_SCHEDULE=0 */6 * * * # cron format, runs a test every 6 hours
      - DISPLAY_TIMEZONE=America/New_York #optional
      - PRUNE_RESULTS_OLDER_THAN=0 #optional, days to keep results (0 = keep forever)
    volumes:
      - /mnt/tank/configs/speedtesttracker:/config
```

> Generate your own `APP_KEY` with `echo "base64:$(openssl rand -base64 32)"` and paste the full output, `base64:` prefix included. Set `APP_URL` to the address you'll use to reach the app. Without `SPEEDTEST_SCHEDULE`, tests only run when you start them by hand.
{.is-warning}
# 2 · First Login
The default user is `admin@example.com` and the default password is `password`. Change both right after your first login.

> Find additional environment variables in their [documentation](https://docs.speedtest-tracker.dev/getting-started/environment-variables) 
{.is-info}
