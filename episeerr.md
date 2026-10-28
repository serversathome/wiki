---
title: Episeerr
description: A guide to deploying Episeerr
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-20T20:27:15.149Z
---

# What is Episeerr?

**Episeerr** is a smart episode management tool for Sonarr that automates your TV library with granular episode-level control. It lets you select exactly which episodes to download, automatically queues the next episode as you watch, and intelligently cleans up old content when storage runs low. It's perfect for limited storage setups like seedboxes, VPS, or budget home servers where you need to prevent runaway downloads from filling your disk.

Episeerr integrates with **Sonarr**, **Plex**, **Tautulli**, **Jellyfin** or **Emby** (for watch tracking), **Seerr** (for request management), and **TMDB** (for metadata).

<div class="glance">
  <div><span>Port</span><b><code>5002</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Episeerr

```yaml
services:
  episeerr:
    image: vansmak/episeerr:latest
    container_name: episeerr
    environment:
      # Required (or leave these out and configure everything at http://your-server:5002/setup)
      - SONARR_URL=http://your-sonarr:8989
      - SONARR_API_KEY=your_sonarr_api_key
      - TMDB_API_KEY=your_tmdb_api_key
      # Optional - Jellyseerr/Overseerr (episode-level request management)
      - JELLYSEERR_URL=http://your-jellyseerr-url
      - JELLYSEERR_API_KEY=your_jellyseerr_api_key
      # Optional - Viewing automation (pick ONE)
      # Plex native webhooks (recommended for Plex, requires Plex Pass)
      # - PLEX_URL=http://your-plex:32400
      # - PLEX_TOKEN=your_plex_token
      # Or Tautulli (don't combine with Plex native webhooks)
      - TAUTULLI_URL=http://your-tautulli:8181
      - TAUTULLI_API_KEY=your_tautulli_key
      # Or Jellyfin
      - JELLYFIN_URL=http://your-jellyfin-url
      - JELLYFIN_API_KEY=your_jf_key
      - JELLYFIN_USER_ID=your_jf_user_id
      # Or Emby
      # - EMBY_URL=http://your-emby:8096
      # - EMBY_API_KEY=your_emby_api_key
      # - EMBY_USER_ID=your_username
      # Optional - Password protection (recommended if exposed beyond your LAN)
      # - REQUIRE_AUTH=true
      # - AUTH_USERNAME=admin
      # - AUTH_PASSWORD=your-password-here
      # - SECRET_KEY=  # auto-generated if not set; or use: openssl rand -hex 32
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
> You will need to configure **webhooks** in Sonarr (required), and optionally in Plex, Tautulli, Jellyfin or Emby if you want viewing automation.
{.is-warning}

# 2 · Configuration

## 2.1 Initial Setup

After deploying the container, open the setup page at `http://your-server:5002/setup`. Everything (Sonarr, TMDB, your media server and Seerr) can be configured there, so environment variables are optional. If you do set them, they show up as defaults on the setup page, and values saved in the GUI take priority.

## 2.2 Sonarr Webhook

You **must** add a webhook in Sonarr pointing to Episeerr for the integration to work.

1. In Sonarr, go to **Settings** → **Connect**
2. Add a new **Webhook**
3. Set the URL to `http://your-episeerr:5002/sonarr-webhook` with method **POST**
4. Enable only the **On Series Add** and **On Grab** triggers

## 2.3 Viewing Automation Webhooks

Pick **one** watch source. Configure it on the `/setup` page under **Integrations**, then add the matching webhook:

| Source | Webhook URL |
|--------|-------------|
| Plex (native, Plex Pass) | `http://your-episeerr:5002/api/integration/plex/webhook` |
| Tautulli | `http://your-episeerr:5002/api/integration/tautulli/webhook` (legacy `/webhook` still works) |
| Jellyfin | `http://your-episeerr:5002/api/integration/jellyfin/webhook` |
| Emby | `http://your-episeerr:5002/api/integration/emby/webhook` |

> Don't run Plex native webhooks and a Tautulli "Watched" webhook at the same time. Both trigger episode processing, which causes double downloads.
{.is-warning}

## 2.4 Ways to Use Episeerr

Episeerr is modular — you can use as little or as much as you need:

| Mode | What It Does | Requirements |
|------|-------------|--------------|
| Episode Selection | Manually pick exactly which episodes to download, even across seasons | Sonarr + TMDB (required env vars only) |
| Viewing Automation | Next episode auto-queues as you watch, keep X episodes | Add a Plex, Tautulli, Jellyfin or Emby webhook + create rules |
| Storage Management | Automatic cleanup when disk space runs low | Set storage threshold + add grace/dormant timers to rules |
| Plex Watchlist Sync | Add a show to your Plex watchlist and Episeerr creates the selection request | Plex connected on `/setup` with Watchlist Auto-Sync enabled |
| Always Have | Baseline episodes (e.g. pilots) that are always kept and protected from Grace/Keep cleanup | An Always Have expression (e.g. `s1e1`) in the rule |


## 2.5 Smart Rules

Create rules using the dropdown system in the UI to control how episodes are managed:

**Get Episodes** — how many episodes to queue ahead. Example: "3 episodes" means the next 3 episodes will be ready to watch.

**Keep Episodes** — how many watched episodes to retain. Example: "1 season" means keep the current season after watching.

## 2.6 Grace Periods & Dormant Timer

**Grace Watched** — kept episodes expire after X days of inactivity. Example: 14 days means watched episodes rotate out after 2 weeks.

**Grace Unwatched** — new episodes get X days to be watched before cleanup. Example: 10 days creates a deadline to watch new content.

**Dormant Timer** — removes content from shows with no activity. Example: 30 days means if nobody watches for a month, the show gets cleaned up.

## 2.7 Storage Gate

The Storage Gate is a global threshold you set (e.g., "Keep 20GB free") that controls when cleanup runs:

- Cleanup **only** triggers when storage drops below the threshold
- Stops immediately once storage is back above the threshold
- Only affects shows that have grace or dormant timers configured

> 
> The Storage Gate prevents unnecessary cleanup — episodes are only removed when you actually need the space.
{.is-success}

## 2.8 Sonarr "Watched" Tag

Adding a `watched` tag to a series in Sonarr will remove it from Episeerr's Series Management view, keeping your dashboard clean for actively managed shows.


# <img src="/patreon-light.png" class="tab-icon"> 3 · Video
[![](/2026-02-10-automatic-episode-management-wit-promo-card.png)](https://www.patreon.com/posts/automatic-with-148716503)