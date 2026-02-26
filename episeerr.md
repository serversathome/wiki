---
title: Episeerr
description: A guide to deploying Episeerr
published: true
date: 2026-02-26T16:22:05.672Z
tags: 
editor: markdown
dateCreated: 2026-01-20T20:27:15.149Z
---

# What is Episeerr?

**Episeerr** is a smart episode management tool for Sonarr that automates your TV library with granular episode-level control. It lets you select exactly which episodes to download, automatically queues the next episode as you watch, and intelligently cleans up old content when storage runs low. It's perfect for limited storage setups like seedboxes, VPS, or budget home servers where you need to prevent runaway downloads from filling your disk.

Episeerr integrates with **Sonarr**, **Tautulli** or **Jellyfin** (for watch tracking), **Seerr** (for request management), and **TMDB** (for metadata).

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Episeerr

```yaml
services:
  episeerr:
    image: vansmak/episeerr:latest
    container_name: episeerr
    environment:
      # Required
      - SONARR_URL=http://your-sonarr:8989
      - SONARR_API_KEY=your_sonarr_api_key
      - TMDB_API_KEY=your_tmdb_api_key
      # Optional - Jellyseerr/Overseerr (episode-level request management)
      - JELLYSEERR_URL=http://your-jellyseerr-url
      - JELLYSEERR_API_KEY=your_jellyseerr_api_key
      # Optional - Viewing automation (pick one)
      - TAUTULLI_URL=http://your-tautulli:8181
      - TAUTULLI_API_KEY=your_tautulli_key
      # Or Jellyfin
      - JELLYFIN_URL=http://your-jellyfin-url
      - JELLYFIN_API_KEY=your_jf_key
      - JELLYFIN_USER_ID=your_jf_user_id
      # Optional - Custom quicklinks
      - CUSTOMAPP_URL=http://192.168.1.100:8080
      - CUSTOMAPP_NAME=My Custom App
      - CUSTOMAPP_ICON=fas fa-cog
    volumes:
      - /mnt/tank/configs/episeerr/config:/app/config
      - /mnt/tank/configs/episeerr/logs:/app/logs
      - /mnt/tank/configs/episeerr/data:/app/data
      - /mnt/tank/configs/episeerr/temp:/app/temp
    ports:
      - "5002:5002"
    restart: unless-stopped
```

> 
> The `JELLYSEERR_URL` field name should be used even if you are running Seerr — keep the field name as `JELLYSEERR_URL`.
{.is-info}

> 
> You will need to configure **webhooks** in Sonarr (required), and optionally in Tautulli or Jellyfin if you want viewing automation.
{.is-warning}

# 2 · Configuration

## 2.1 Initial Setup

After deploying the container, open the Episeerr web UI at `http://your-server:5002`. At minimum, you need three environment variables configured to get started: `SONARR_URL`, `SONARR_API_KEY`, and `TMDB_API_KEY`.

## 2.2 Sonarr Webhook

You **must** add a webhook in Sonarr pointing to Episeerr for the integration to work.

1. In Sonarr, go to **Settings** → **Connect**
2. Add a new **Webhook**
3. Set the URL to `http://your-episeerr:5002`

## 2.3 Three Ways to Use Episeerr

Episeerr is modular — you can use as little or as much as you need:

| Mode | What It Does | Requirements |
|------|-------------|--------------|
| Episode Selection | Manually pick exactly which episodes to download, even across seasons | Sonarr + TMDB (required env vars only) |
| Viewing Automation | Next episode auto-queues as you watch, keep X episodes | Add Tautulli or Jellyfin webhook + create rules |
| Storage Management | Automatic cleanup when disk space runs low | Set storage threshold + add grace/dormant timers to rules |


## 2.4 Smart Rules

Create rules using the dropdown system in the UI to control how episodes are managed:

**Get Episodes** — how many episodes to queue ahead. Example: "3 episodes" means the next 3 episodes will be ready to watch.

**Keep Episodes** — how many watched episodes to retain. Example: "1 season" means keep the current season after watching.

## 2.5 Grace Periods & Dormant Timer

**Grace Watched** — kept episodes expire after X days of inactivity. Example: 14 days means watched episodes rotate out after 2 weeks.

**Grace Unwatched** — new episodes get X days to be watched before cleanup. Example: 10 days creates a deadline to watch new content.

**Dormant Timer** — removes content from shows with no activity. Example: 30 days means if nobody watches for a month, the show gets cleaned up.

## 2.6 Storage Gate

The Storage Gate is a global threshold you set (e.g., "Keep 20GB free") that controls when cleanup runs:

- Cleanup **only** triggers when storage drops below the threshold
- Stops immediately once storage is back above the threshold
- Only affects shows that have grace or dormant timers configured

> 
> The Storage Gate prevents unnecessary cleanup — episodes are only removed when you actually need the space.
{.is-success}

## 2.7 Sonarr "Watched" Tag

Adding a `watched` tag to a series in Sonarr will remove it from Episeerr's Series Management view, keeping your dashboard clean for actively managed shows.


# <img src="/patreon-light.png" class="tab-icon"> 3 · Video

*Coming Soon*