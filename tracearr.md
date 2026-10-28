---
title: Tracearr
description: A guide to deploying Tracearr
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:09:01.316Z
---

# <img src="/tracearr.png" class="tab-icon"> What is Tracearr?

**Tracearr** is a real-time monitoring platform for **Plex**, **Jellyfin**, and **Emby** servers. It combines session tracking, playback analytics, library insights, and account sharing detection into a single dashboard — replacing the need to run separate tools like Tautulli and Jellystat for each media server.

Key features include stream geolocation mapping, trust scoring, impossible travel detection, concurrent stream limits, Discord webhook alerts, and a public REST API. Tracearr can also import existing watch history from Tautulli and Jellystat so you don't start from scratch.

<div class="glance">
  <div><span>Port</span><b><code>3000</code></b></div>
  <div><span>Deploy via</span><b>TrueNAS app or Docker compose</b></div>
  <div><span>Containers</span><b>3 services (or 1 all-in-one)</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/connorgallopo/tracearr"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# 1 · Deploy Tracearr
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

The recommended upstream deployment runs Tracearr with a separate TimescaleDB (PostgreSQL 18) and Redis container.

```yaml
services:
  tracearr:
    image: ghcr.io/connorgallopo/tracearr:latest
    container_name: tracearr
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - HOST=0.0.0.0
      - TZ=America/New_York
      - DATABASE_URL=postgres://tracearr:CHANGE_ME_DB_PASSWORD@timescale:5432/tracearr
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=CHANGE_ME # generate with: openssl rand -hex 32
      - COOKIE_SECRET=CHANGE_ME # generate with: openssl rand -hex 32
      - LOG_LEVEL=info
    depends_on:
      timescale:
        condition: service_healthy
      redis:
        condition: service_healthy
    volumes:
      - /mnt/tank/configs/tracearr/backup:/data/backup
      - /mnt/tank/configs/tracearr/image-cache:/data/tracearr/image-cache
    restart: unless-stopped

  timescale:
    image: timescale/timescaledb-ha:pg18.4-ts2.29.1
    container_name: tracearr-db
    shm_size: 512mb
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
    command: postgres -c timescaledb.license=timescale -c timescaledb.max_tuples_decompressed_per_dml_transaction=100000 -c max_locks_per_transaction=4096 -c timescaledb.telemetry_level=off -c max_connections=150
    environment:
      - POSTGRES_USER=tracearr
      - POSTGRES_PASSWORD=CHANGE_ME_DB_PASSWORD
      - POSTGRES_DB=tracearr
    volumes:
      - /mnt/tank/configs/tracearr/db:/home/postgres/pgdata/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U tracearr"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  redis:
    image: redis:8-alpine
    container_name: tracearr-redis
    command: redis-server --appendonly yes
    volumes:
      - /mnt/tank/configs/tracearr/redis:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped
```

1. Generate the two secrets with `openssl rand -hex 32` (run it twice, one value each for `JWT_SECRET` and `COOKIE_SECRET`)
2. Replace `CHANGE_ME_DB_PASSWORD` in both places with the same strong password
3. Adjust `TZ` to match your timezone
4. Deploy the stack and open `http://YOUR_SERVER_IP:3000`

> The `timescaledb-ha` image runs as UID 1000, so make sure `/mnt/tank/configs/tracearr/db` is owned by UID 1000 before deploying. PostgreSQL 18 data is not compatible with PostgreSQL 15/16, so point this at a new, empty folder.
{.is-info}

### All-in-one (supervised) image

The `supervised` image bundles Tracearr, TimescaleDB, and Redis into a single container, and secrets are auto-generated on first run. Upstream now describes it as **designed for Unraid and bare-metal hosts only**, and not recommended for VMs or nested containers.

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
      - tracearr_postgres:/data/postgres
      - tracearr_redis:/data/redis
      - tracearr_data:/data/tracearr
      - tracearr_backups:/data/backup
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://127.0.0.1:3000/health"]
      interval: 30s
      timeout: 10s
      start_period: 60s
      retries: 3

volumes:
  tracearr_postgres:
  tracearr_redis:
  tracearr_data:
  tracearr_backups:
```

> The supervised image requires a minimum of **3 GB of RAM** (`mem_limit`) since it bundles PostgreSQL (TimescaleDB), Redis, and Node.js in one container. The `shm_size` and `ulimits` settings are required for TimescaleDB to function properly. Upstream recommends Docker-managed volumes here, as shown, because bind mounts cause permission issues with the bundled PostgreSQL and Redis.
{.is-info}

> **Upgrading an existing all-in-one install?** Earlier versions of this page used `/mnt/tank/configs/tracearr/*` bind mounts. If yours did, keep those mounts instead of switching to the named volumes above, or Tracearr starts empty and your history stays behind in the old folders. Use these lines under `volumes:` and leave out the top-level `volumes:` block:
> ```yaml
>       - /mnt/tank/configs/tracearr/postgres:/data/postgres
>       - /mnt/tank/configs/tracearr/redis:/data/redis
>       - /mnt/tank/configs/tracearr/data:/data/tracearr
>       - /mnt/tank/configs/tracearr/backup:/data/backup
> ```
> The `backup` folder is new: create it and make it owned by UID 1001 (the image's `tracearr` user), for example `mkdir -p /mnt/tank/configs/tracearr/backup && chown 1001 /mnt/tank/configs/tracearr/backup`.
{.is-warning}


### Docker Tags

| Tag | Description |
|-----|-------------|
| `latest` | Stable release (requires external DB/Redis) |
| `supervised` | All-in-one stable release (bundles DB + Redis) |
| `next` | Latest prerelease (requires external DB/Redis) |
| `supervised-next` | All-in-one prerelease |
| `nightly` | Bleeding edge nightly build (requires external DB/Redis) |
| `supervised-nightly` | All-in-one nightly build |

## <img src="/truenas.png" class="tab-icon"> TrueNAS

Tracearr is available in the **Community** train of the TrueNAS Apps catalog.

1. Navigate to **Apps** in the TrueNAS UI
2. Click **Discover Apps**
3. Search for **Tracearr**
4. Click **Install**
5. Configure the following settings:
   - **Timezone**: Set to your local timezone
   - **Database Password**: Set a strong password
   - **Redis Password**: Set a strong password
   - **JWT Secret**: JWT signing key. Generate with: `openssl rand -hex 32`
   - **Cookie Secret**: Cookie signing key. Generate with: `openssl rand -hex 32`
   - **Web Port**: `3000` (default)
   - **Postgres Data Storage**: Set to `Host Path` and check the box for **Automatic Permissions**!
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


## 2.3 Automations (Sharing Detection)

Sharing detection now runs through **Automations**: a trigger fires, conditions decide whether it matters, and actions run (send a notification, adjust trust, message the client, or stop the stream). Tracearr ships 22 built-in templates, including these sharing rules:

| Rule | Description |
|------|-------------|
| Impossible Travel | Flags sessions from two distant locations in an impossibly short time |
| Simultaneous Locations | Same account streaming from two cities at once |
| Device Velocity | Too many unique IPs in a short window |
| Concurrent Streams | Per-user stream limits |
| Geo Restrictions | Block streaming from specific countries |
| Account Inactivity | Alerts when accounts go dormant |

## 2.4 Notifications

Configure these under **Settings > Notifications** to receive real-time alerts when an automation sends a notification.

## 2.5 Data Import

If you're migrating from **Tautulli** or **Jellystat**, you can import your existing watch history under **Settings > Import**. This preserves your historical data so you don't lose any analytics.


> Large imports from Tautulli may take some time due to TimescaleDB chunk processing.
{.is-warning}

## 2.6 Public API

Tracearr exposes a read-only REST API for third-party integrations. Generate an API key under **Settings**, then explore available endpoints at `http://YOUR_SERVER_IP:3000/api-docs` (Swagger UI). This works with Homarr, Home Assistant, or any HTTP-compatible tool.





# <img src="/patreon-light.png" class="tab-icon"> 3 · Video
[![](/2025-12-22-track-user-violations-with-trace-promo-card.png)](https://www.patreon.com/posts/track-user-with-146451791)