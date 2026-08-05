---
title: JellyGlance
description: A guide to deploy JellyGlance
published: true
date: 2026-08-05T20:28:45.437Z
tags: 
editor: markdown
dateCreated: 2026-08-05T20:28:45.437Z
---

# What is JellyGlance?

**JellyGlance** is a modern dashboard and command center for Jellyfin. It pulls live playback sessions, user watch statistics, recently added media, library health, release calendars, download queues, and media requests into one interface so you stop juggling a dozen browser tabs to see what your media server is doing.

It goes further than a pure stats dashboard. JellyGlance also acts as an operations layer for your whole media stack: it connects to the Arr apps, Jellyseerr/Overseerr, and your download clients, then surfaces integration health, webhook delivery history, an admin audit log, scheduled tasks, and backup freshness in one place.

The project is inspired by **Jellystat**, but adds request triage, per-user profile pages, a customizable home layout with kiosk mode, and a much larger integration list.



# <img src="/docker.png" class="tab-icon"> 1 · Deploy JellyGlance

```yaml
services:
  jellyglance-db:
    image: postgres:16-alpine
    container_name: jellyglance-db
    restart: unless-stopped
    shm_size: "1gb"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: change-me
      POSTGRES_DB: jellyglance
    volumes:
      - /mnt/tank/configs/jellyglance/postgres:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready --dbname=jellyglance --username=postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  jellyglance:
    image: ghcr.io/nerdy-technician/jellyglance:latest
    container_name: jellyglance
    restart: unless-stopped
    depends_on:
      jellyglance-db:
        condition: service_healthy
    ports:
      - "3000:3000"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: change-me
      POSTGRES_IP: jellyglance-db
      POSTGRES_PORT: 5432
      POSTGRES_DB: jellyglance
      JWT_SECRET: replace-me-with-a-long-random-secret
      TZ: America/New_York
      CONFIG_DIR: /app/config
      BACKUP_DIR: /app/backups
    volumes:
      - /mnt/tank/configs/jellyglance/config:/app/config
      - /mnt/tank/configs/jellyglance/backups:/app/backups
```

1. Change `POSTGRES_PASSWORD` in **both** services — they have to match or the app will not connect.
2. Replace `JWT_SECRET` with a long random string. Generate one with `openssl rand -base64 48`.
3. Set `TZ` to your timezone so timestamps and scheduled tasks line up.


# 2 · Configuration

## 2.1 First Run

On first launch you are dropped into a setup wizard:

1. Enter your **Jellyfin server URL** (for example `http://192.168.1.50:8096`).
2. Enter a **Jellyfin API key**. Generate one in Jellyfin under **Dashboard → Advanced → API Keys**.
3. Choose your **admin access mode**.
4. Complete Jellyfin Quick Connect, OIDC provider details, or create a local admin account.
5. Let the first sync finish before poking around — it pulls users, libraries, artwork, sessions, and activity history.

> Once the sync completes, JellyGlance can pull cached artwork from your Jellyfin library to use as the login background and throughout the media views.
{.is-info}

## 2.2 Connecting Integrations

Everything else lives under **Settings → Integrations**. Add each service with its URL and API key:

| Type | Supported Apps |
|------|----------------|
| Media server | Jellyfin |
| Arr apps | Sonarr, Radarr, Lidarr, Bazarr |
| Seerr apps | Jellyseerr, Overseerr |
| Download clients | qBittorrent, Transmission, Deluge, SABnzbd, NZBGet |
| Auth | Jellyfin Quick Connect, local accounts, OIDC-ready flow |
| Notifications | Discord-compatible webhooks, Gotify-style webhooks |


Adding the Arr apps lights up the **Calendar** page with upcoming releases. Adding a download client lights up the **Downloads** page with active queues, plus torrent URL and magnet link submission.


## 2.3 Customizing the Home Page

The home dashboard is fully rearrangeable. Drag sections into the order you want, hide the widgets you do not care about, pick a role-focused preset, and switch between compact and comfortable density.

For a wall-mounted display, open `/home/kiosk` to get a sidebar-free view.

