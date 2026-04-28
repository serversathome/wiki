---
title: Prismarr
description: A guide to deploying Prismarr
published: true
date: 2026-04-28T00:21:28.596Z
tags: 
editor: markdown
dateCreated: 2026-04-28T00:21:28.596Z
---

# What is Prismarr?

**Prismarr** is a self-hosted media dashboard that unifies Radarr, Sonarr, Prowlarr, Jellyseerr, qBittorrent and TMDb into a single Symfony 8 interface. Instead of juggling six browser tabs to manage your library, Prismarr gives you one search bar across the local library and TMDb, one calendar that merges movie and episode releases, one dashboard that surfaces what matters today, and one settings page where every API key lives.

It's not a replacement for your existing \*arr stack — Radarr and Sonarr keep doing what they do best. Prismarr sits on top as the unified control surface, consuming the APIs of services you already run. The whole thing ships as a single Docker container with embedded SQLite (no external database, no Redis, no per-service `.env` files), is multi-arch (amd64 + arm64), and is licensed AGPL-3.0.

# 1 · Deploy Prismarr

```yaml
services:
  prismarr:
    image: shoshuo/prismarr:latest
    container_name: prismarr
    restart: unless-stopped
    stop_grace_period: 30s
    ports:
      - "7070:7070"
    volumes:
      - /mnt/tank/configs/prismarr:/var/www/html/var/data
```


# 2 · Configuration

## 2.1 Setup Wizard

The first time you open Prismarr, the wizard at `/setup` walks you through the seven steps in order. You can skip optional services (TMDb, qBittorrent, Gluetun) and add them later from the admin settings page.

Every API key, password, and service URL is stored in the SQLite database (`setting` table) — never in environment variables, never in any committable file. The export feature strips every key matching `api_key`, `password`, or `secret`, so sharing your config is safe.

## 2.2 Admin Settings

Once setup is complete, head to `/admin/settings` (admin only) to manage:

- **Services** — add, remove, or update Radarr / Sonarr / Prowlarr / Jellyseerr / qBittorrent / TMDb credentials and URLs
- **Display preferences** — theme color, UI density, toasts, timezone, date/time format, qBittorrent auto-refresh interval, default home page
- **Languages** — English (source) and French (full ICU plural support); easy to add more by duplicating the YAML file and translating
- **Backup** — export the full settings JSON (credentials stripped) for portability

## 2.3 Reverse Proxy

Prismarr emits its own HSTS and Permissions-Policy headers via the embedded Caddy server. When sitting behind a reverse proxy that terminates TLS, set the `TRUSTED_PROXIES` environment variable to your proxy network so Symfony reads the correct `X-Forwarded-*` headers.

```yaml
environment:
  - TRUSTED_PROXIES=10.0.0.0/8,172.16.0.0/12,192.168.0.0/16
```



# <img src="/patreon-light.png" class="tab-icon"> 3 · Video

*Video coming soon — check back later!*