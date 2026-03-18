---
title: Tracearr
description: A guide to deploying Tracearr
published: true
date: 2026-03-18T20:57:45.337Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:09:01.316Z
---

# <img src="/tracearr.png" class="tab-icon"> What is Tracearr?

**Tracearr** is a real-time monitoring platform for **Plex**, **Jellyfin**, and **Emby** servers. It combines session tracking, playback analytics, library insights, and account sharing detection into a single dashboard — replacing the need to run separate tools like Tautulli and Jellystat for each media server.

Key features include stream geolocation mapping, trust scoring, impossible travel detection, concurrent stream limits, Discord webhook alerts, and a public REST API. Tracearr can also import existing watch history from Tautulli and Jellystat so you don't start from scratch.

# 1 · Deploy Tracearr
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

The `supervised` image bundles Tracearr, TimescaleDB, and Redis into a single container with zero configuration — secrets are auto-generated on first run.

```yaml
services:
  tracearr:
    image: ghcr.io/connorgallopo/tracearr:supervised
    container_name: tracearr
    shm_size: 512mb
    mem_limit: 3g
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
    ports:
      - "3000:3000"
    environment:
      - TZ=America/New_York
      - LOG_LEVEL=info
    volumes:
      - /mnt/tank/configs/tracearr/postgres:/data/postgres
      - /mnt/tank/configs/tracearr/redis:/data/redis
      - /mnt/tank/configs/tracearr/data:/data/tracearr
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://127.0.0.1:3000/health"]
      interval: 30s
      timeout: 10s
      start_period: 60s
      retries: 3
```

1. Adjust `TZ` to match your timezone
2. Deploy the stack and open `http://YOUR_SERVER_IP:3000`


> The supervised image requires a minimum of **3 GB of RAM** (`mem_limit`) since it bundles PostgreSQL (TimescaleDB), Redis, and Node.js in one container. The `shm_size` and `ulimits` settings are required for TimescaleDB to function properly.
{.is-info}


### Docker Tags

| Tag | Description |
|-----|-------------|
| `supervised` | All-in-one stable release (bundles DB + Redis) |
| `latest` | Stable release (requires external DB/Redis) |
| `supervised-next` | All-in-one prerelease |
| `nightly` | Bleeding edge nightly build |

## <img src="/truenas.png" class="tab-icon"> TrueNAS

Tracearr is available in the **Community** train of the TrueNAS Apps catalog.

1. Navigate to **Apps** in the TrueNAS UI
2. Click **Discover Apps**
3. Search for **Tracearr**
4. Click **Install**
5. Configure the following settings:
   - **Timezone**: Set to your local timezone
   - **Web Port**: `3000` (default)
6. Click **Save** and wait for the app to deploy


> Make sure you have the **Community** train enabled under **Apps > Settings > Preferred Trains** to see Tracearr in the catalog.
{.is-info}

# 2 · Configuration

## 2.1 Initial Setup

When you first open Tracearr at `http://YOUR_SERVER_IP:3000`, you'll be guided through the initial setup wizard to create your admin account.

## 2.2 Connect Your Media Servers

After logging in, navigate to **Settings** and connect your media servers:

1. Click **Add Server**
2. Select the server type: **Plex**, **Jellyfin**, or **Emby**
3. Enter the server URL and authentication details
4. Click **Save**

You can connect multiple servers of different types — Tracearr will display them all in a unified dashboard.


## 2.3 Sharing Detection Rules

Tracearr includes six types of account sharing detection rules you can configure under **Settings > Sharing Rules**:

| Rule | Description |
|------|-------------|
| Impossible Travel | Flags sessions from two distant locations in an impossibly short time |
| Simultaneous Locations | Same account streaming from two cities at once |
| Device Velocity | Too many unique IPs in a short window |
| Concurrent Streams | Per-user stream limits |
| Geo Restrictions | Block streaming from specific countries |
| Account Inactivity | Alerts when accounts go dormant |

## 2.4 Notifications

Configure these under **Settings > Notifications** to receive real-time alerts when sharing detection rules are triggered.

## 2.5 Data Import

If you're migrating from **Tautulli** or **Jellystat**, you can import your existing watch history under **Settings > Import**. This preserves your historical data so you don't lose any analytics.


> Large imports from Tautulli may take some time due to TimescaleDB chunk processing.
{.is-warning}

## 2.6 Public API

Tracearr exposes a read-only REST API for third-party integrations. Generate an API key under **Settings**, then explore available endpoints at `http://YOUR_SERVER_IP:3000/api-docs` (Swagger UI). This works with Homarr, Home Assistant, or any HTTP-compatible tool.





# <img src="/patreon-light.png" class="tab-icon"> 3 · Video
[![](/2025-12-22-track-user-violations-with-trace-promo-card.png)](https://www.patreon.com/posts/track-user-with-146451791)