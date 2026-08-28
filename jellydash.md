---
title: Jellydash
description: A guide to deploying Jellydash
published: true
date: 2026-08-28T20:28:41.792Z
tags: 
editor: markdown
dateCreated: 2026-08-28T20:28:41.792Z
---

# <img src="/jellydash.png" class="tab-icon"> What is Jellydash?

**Jellydash** is a self-hosted monitoring dashboard for your Jellyfin server. If you have used Tautulli on the Plex side, this is the same idea built for Jellyfin: live sessions, complete play history, statistics, library overviews, optional Jellyseerr requests, and push notifications when someone hits play.





# <img src="/docker.png" class="tab-icon"> 1 · Deploy Jellydash

Create the config directories first so the bind mounts do not come up as root-owned:

```bash
mkdir -p /mnt/tank/configs/jellydash/{cache,runtime-cache,logs,uploads,db}
chown -R 33:33 /mnt/tank/configs/jellydash/{cache,runtime-cache,logs,uploads}
```

Then deploy the stack:

```yaml
services:
  jellydash:
    image: ghcr.io/themartz90/jellydash:latest
    container_name: jellydash
    restart: unless-stopped
    ports:
      - "8080:80"
    environment:
      APP_ENV: production
      APP_DEBUG: "false"
      APP_TIMEZONE: America/New_York
      APP_URL: ""
      DB_HOST: jellydash-db
      DB_PORT: "3306"
      DB_DRIVER: mysqli
      DB_NAME: jellydash
      DB_USER: jellydash
      DB_PASS: CHANGE_ME
      JELLYFIN_URL: http://192.168.1.10:8096
      JELLYFIN_API_TOKEN: CHANGE_ME
      JELLYFIN_VERIFY_SSL: "true"
      POLLER_ENABLED: "true"
      POLL_INTERVAL: "30"
      PUSH_ENABLED: "true"
      AUTH_ENABLED: "false"
    depends_on:
      jellydash-db:
        condition: service_healthy
    volumes:
      - /mnt/tank/configs/jellydash/cache:/var/www/html/cache
      - /mnt/tank/configs/jellydash/runtime-cache:/var/www/html/var/cache
      - /mnt/tank/configs/jellydash/logs:/var/www/html/var/log
      - /mnt/tank/configs/jellydash/uploads:/var/www/html/public/uploads

  jellydash-db:
    image: mariadb:11
    container_name: jellydash-db
    restart: unless-stopped
    environment:
      MARIADB_RANDOM_ROOT_PASSWORD: "yes"
      MARIADB_DATABASE: jellydash
      MARIADB_USER: jellydash
      MARIADB_PASSWORD: CHANGE_ME
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 5s
      timeout: 5s
      retries: 12
    volumes:
      - /mnt/tank/configs/jellydash/db:/var/lib/mysql
```

1. Replace both instances of `DB_PASS` / `MARIADB_PASSWORD` with the same strong password
2. Point `JELLYFIN_URL` at your Jellyfin server
3. Drop in your Jellyfin API token (see **2.1** below)


# 2 · Configuration

## 2.1 Jellyfin API Key

1. In Jellyfin, go to **Dashboard → API Keys**
2. Click **+** and name the key `Jellydash`
3. Copy the key into `JELLYFIN_API_TOKEN`

> Use an **admin-level** key. Some statistics need to read library paths and will come back empty otherwise.
{.is-warning}

If your Jellyfin runs behind self-signed HTTPS, set `JELLYFIN_VERIFY_SSL=false`.

## 2.2 Background Poller

`POLLER_ENABLED=true` is what makes history complete. The poller checks Jellyfin every `POLL_INTERVAL` seconds (30 is a good balance) and records plays even when nobody has the dashboard open. It also warms the cache and fires the notifications.

Leave this on. With it off you only see what happens while the page is loaded.

## 2.3 Jellyseerr (Optional)

Add these to the `jellydash` service and the Jellyseerr page appears in the menu:

```yaml
      JELLYSEER_URL: http://192.168.1.10:5055
      JELLYSEER_API_TOKEN: CHANGE_ME
      JELLYSEER_VERIFY_SSL: "true"
      SEERR_POLL_INTERVAL: "120"
      SEERR_NOTIFY_ENABLED: "true"
```

The API key comes from **Jellyseerr → Settings → General → API Key**. Note the env var spelling is `JELLYSEER_` with one **R**.

