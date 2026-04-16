---
title: Reclaimerr
description: A guide to deploying Reclaimerr
published: true
date: 2026-04-16T20:23:27.413Z
tags: 
editor: markdown
dateCreated: 2026-04-16T20:23:27.413Z
---

# <img src="/reclaimerr.png" class="tab-icon"> What is Reclaimerr?

**Reclaimerr** is a self-hosted tool that helps you automatically identify and clean up unwatched, low-rated, or stale media in your Jellyfin and/or Plex library so you can reclaim disk space. Think of it as Seerr in reverse — instead of requesting new media, you're cleaning up what's already there. Reclaimerr supports rule-based filtering, a protection system to safeguard favorites, optional Sonarr/Radarr/Seerr integration, and notifications through Apprise.

> 
> Reclaimerr is currently in **beta**. While in beta, automatic deletion is intentionally disabled — deletions must be processed manually through the UI or API. Automatic deletion will be added as an opt-in feature once the project matures.
{.is-warning}

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Reclaimerr

Reclaimerr requires both a compose file and an accompanying `.env` file for its environment variables.

**compose.yaml**

```yaml
services:
  reclaimerr:
    image: ghcr.io/jessielw/reclaimerr:latest
    container_name: reclaimerr
    restart: unless-stopped
    env_file: ".env"
    volumes:
      - /mnt/tank/configs/reclaimerr:/app/data
    ports:
      - "8000:8000"
```

**.env**

```bash
# directory to store application data (database, logs, static files, etc.)
DATA_DIR=./data

# API configuration
API_HOST=0.0.0.0
API_PORT=8000
CORS_ORIGINS=http://localhost:3000

# secrets — leave blank to auto-generate stable values on first launch (recommended),
# or set your own (min 32 characters, e.g. `openssl rand -hex 32`)
# JWT_SECRET=
# ENCRYPTION_KEY=

# logging (options: DEBUG, INFO, WARNING, ERROR, CRITICAL)
# LOG_LEVEL=INFO

# set to true when serving over HTTPS
# COOKIE_SECURE=false
```


> 
> Leave `JWT_SECRET` and `ENCRYPTION_KEY` **blank** on first launch. Reclaimerr will auto-generate stable values for you. Only set them manually if you have a specific reason to.
{.is-info}

> 
> Update `CORS_ORIGINS` to match the URL you'll use to access Reclaimerr. If you're serving it behind a reverse proxy with a custom domain, point this at that domain.
{.is-info}

# 2 · Configuration

## 2.1 Media Server Setup

Reclaimerr supports both **Jellyfin** and **Plex** — and you can connect both at the same time.

- Designate one server as the **main** server (source of truth for the library).
- The other server is used as a supplemental data source for watch history.
- Both servers **must** be managing the same physical media library.

Navigate to **Settings → Servers** to add your media server(s). You'll need:

| Field | Value |
|-------|-------|
| Server Type | Jellyfin or Plex |
| URL | `http://<server-ip>:<port>` |
| API Key / Token | From your media server's settings |


## 2.2 Sonarr, Radarr & Seerr Integration

Reclaimerr works **without** Sonarr and Radarr, but integrating them gives you cleaner deletion handling.

When the arrs are configured:
- Deletions are processed through Sonarr/Radarr first (properly unmonitoring/removing the media)
- Falls back to the main media server only if needed
- Optionally removes requests from Jellyseerr or Overseerr to keep your ecosystem in sync

Navigate to **Settings → Integrations** and add your arr stack URLs and API keys.

## 2.3 Rules

Rules define the criteria for what media is considered for reclamation. Examples:

- TV shows that haven't been watched in 12 months
- Movies with a rating below 5.0
- Media last played before a specific date
- Media that has never been watched

Rules are configured through the web UI under **Rules** and can be combined for nuanced filtering.

## 2.4 Media Protection

The protection system prevents specific media from ever being considered for deletion.

- Mark items as protected **permanently** or for a set **duration**
- Users with appropriate permissions can **request** protection on items
- Admins approve or deny protection requests

This is especially useful for shared servers where family or friends have favorites they don't want auto-flagged.

## 2.5 Notifications

Reclaimerr supports **Apprise** for notifications, covering 130+ services including:

- Discord
- Pushover
- ntfy
- Telegram
- Email (SMTP)
- Slack

Configure notifications under **Settings → Notifications** by adding your Apprise URL(s).

## 2.6 Task Scheduling

Scheduled scans are configured via cron expressions or time-based schedules under **Settings → Tasks**. This controls when Reclaimerr scans your library for eligible reclamation candidates.

## 2.7 User Management

Reclaimerr supports multi-user setups with a permission system. Add users under **Settings → Users** and assign permissions for:

- Managing deletions
- Approving protection requests
- Administering servers and rules
- Viewing reports

> 
> Always test your rules on a small subset of your library first. While automatic deletion is disabled during beta, a misconfigured rule can still flag items you'd rather keep — use the protection system liberally until you trust your rules.
{.is-warning}


# <img src="/youtube.png" class="tab-icon"> 4 · Video
