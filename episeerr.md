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

Episeerr integrates with **Sonarr**, **Tautulli** or **Jellyfin** (for watch tracking), **Seerr** (for request management), and **TMDB** (for metadata).

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
> You will need to configure **webhooks** in Sonarr (required), and optionally in Tautulli or Jellyfin if you want viewing automation.
{.is-warning}

# 2 · Configuration

## 2.1 Initial Setup

After deploying the container, open `http://your-server:5002/setup` and enter Sonarr, TMDB, Seerr and your media server there.

## 2.2 Sonarr Webhook

You **must** add a webhook in Sonarr pointing to Episeerr for the integration to work.

1. In Sonarr, go to **Settings** → **Connect**
2. Add a new **Webhook**
3. Set the URL to `http://your-episeerr:5002/sonarr-webhook` with method **POST**
4. Enable only the **On Series Add** and **On Grab** triggers

## 2.3 Viewing Automation Webhooks

| Source | Webhook URL |
|--------|-------------|
| Tautulli | `http://your-episeerr:5002/api/integration/tautulli/webhook` |
| Jellyfin | `http://your-episeerr:5002/api/integration/jellyfin/webhook` |

## 2.4 Three Ways to Use Episeerr

Episeerr is modular — you can use as little or as much as you need:

| Mode | What It Does | Requirements |
|------|-------------|--------------|
| Episode Selection | Manually pick exactly which episodes to download, even across seasons | Sonarr + TMDB |
| Viewing Automation | Next episode auto-queues as you watch, keep X episodes | Add Tautulli or Jellyfin webhook + create rules |
| Storage Management | Automatic cleanup when disk space runs low | Set storage threshold + add grace/dormant timers to rules |


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