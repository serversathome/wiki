---
title: Fetcharr
description: A guide to deploy Fetcharr
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-03-13T15:52:59.553Z
---

# <img src="/fetcharr.png" class="tab-icon"> What is Fetcharr?

**Fetcharr** is a lightweight CLI container that periodically scans your \*arr stack (Radarr, Sonarr, Lidarr, Readarr/Bookshelf, Whisparr) for missing or upgradable media and automatically triggers searches. It was created as a successor to the now-defunct Huntarr project, designed to do one thing well — hunt for better quality releases so you don't have to spend hours manually searching. 

<div class="glance">
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Fetcharr

```yaml
services:
  fetcharr:
    image: egg82/fetcharr:latest
    container_name: fetcharr
    environment:
      - VERIFY_CERTS=false
      - SEARCH_AMOUNT=5
      - SEARCH_INTERVAL=1hour
      - MONITORED_ONLY=true
      - USE_CUTOFF=true
      - RADARR_0_URL=http://radarr:7878
      - RADARR_0_API_KEY=
      - SONARR_0_URL=http://sonar:8989
      - SONARR_0_API_KEY=
    volumes:
      - /mnt/tank/configs/fetcharr/config:/app/config
      - /mnt/tank/configs/fetcharr/cache:/app/cache
      - /mnt/tank/configs/fetcharr/logs:/app/logs
      - /mnt/tank/configs/fetcharr/plugins:/app/plugins
    restart: unless-stopped
```

> Fetcharr runs as UID/GID `1000` inside the container and writes to `/app/config`, `/app/cache`, `/app/logs` and `/app/plugins`. Make sure the host directories are owned by that user: `chown -R 1000:1000 /mnt/tank/configs/fetcharr`. Without this, Sonarr will spam `Could not create parent directory structure` warnings in the logs.
{.is-danger}

> **Upgrading from an older version of this guide?** Fetcharr moved its container paths from `/data` and `/cache` to `/app/config`, `/app/cache`, `/app/logs` and `/app/plugins`. With the old mounts nothing persists. Update your volumes as above and redeploy.
{.is-warning}

> Fetcharr is a **CLI-only** container with no web UI. It runs silently in the background and logs output to the container logs. Use `docker logs fetcharr` to monitor its activity.
{.is-info}

# 2 · Configuration


## 2.1 Common Environment Variables

These settings apply globally to all connected *arr instances unless overridden per-instance:

| Variable | Default | Description |
|----------|---------|-------------|
| `LOG_MODE` | `info` | Logging level: `trace`, `debug`, `info`, `warn`, `error` |
| `SEARCH_AMOUNT` | `5` | Number of items to search per run |
| `SEARCH_INTERVAL` | `1hour` | Time between search runs |
| `MONITORED_ONLY` | `true` | Only search monitored items |
| `USE_CUTOFF` | `false` | Skip items that already meet their quality profile cutoff |
| `MISSING_STATUS` | `any` | `any`, `missing` (missing items only) or `upgrade` (existing items only). Replaces the deprecated `MISSING_ONLY` |
| `SKIP_TAGS` | *(none)* | Comma-separated list of tags to exclude from searches |
| `DRY_RUN` | `false` | Simulate searches without triggering them |
| `CONFIG_DIR` | `/app/config` | Configuration storage directory inside the container |
| `CACHE_DIR` | `/app/cache` | Cache storage directory inside the container |
| `LOG_DIR` | `/app/logs` | Log storage directory inside the container |
| `PUID` / `PGID` | `1000` | Remap the process user/group. Only works when the container is started as root (`user: "0:0"`) |
| `VERIFY_CERTS` | `true` | Verify SSL certificates when connecting to \*arr apps |
| `SSL_PATH` | `/etc/ssl/certs/ca-bundle.crt` | Path to the SSL cert bundle |
| `PROXY_HOST` / `PROXY_PORT` | *(none)* / `80` | HTTP proxy for outgoing requests |
| `CONNECT_TIMEOUT` | `2500` | HTTP connection timeout in milliseconds |
| `REQUEST_TIMEOUT` | `120000` | HTTP request timeout in milliseconds |
| `CONNECT_TTL` | `300000` | HTTP connection TTL in milliseconds |
| `USE_FILE_CACHE` | `auto` | File-based cache: `auto`, `true` or `false` (auto enables it if the cache dir is writable) |
| `USE_MEMORY_CACHE` | `auto` | In-memory cache: `auto`, `true` or `false` |
| `SHORT_CACHE_TIME` | `65minutes` | TTL for short-lived cache entries |
| `LONG_CACHE_TIME` | `6hours` | TTL for long-lived cache entries |
| `MAX_RAM` | `1g` | Upper bound for heap size when the container has no memory limit |
{.dense}

## 2.2 Per-Instance Overrides

Each *arr instance (`RADARR_X_`, `SONARR_X_`, `LIDARR_X_`, `READARR_X_`, `WHISPARR_X_`, where `X` is 0–99) can override the global `SEARCH_AMOUNT`, `SEARCH_INTERVAL`, `MONITORED_ONLY`, `MISSING_STATUS`, `SKIP_TAGS`, and `USE_CUTOFF` settings by using the instance-specific prefix. For example:

```yaml
environment:
  # Global defaults
  - SEARCH_AMOUNT=5
  - SEARCH_INTERVAL=1hour
  # Radarr-specific override — search more movies, less often
  - RADARR_0_URL=https://radarr.yourdomain.com
  - RADARR_0_API_KEY=your-api-key
  - RADARR_0_SEARCH_AMOUNT=10
  - RADARR_0_SEARCH_INTERVAL=2hours
  # Sonarr uses global defaults
  - SONARR_0_URL=https://sonarr.yourdomain.com
  - SONARR_0_API_KEY=your-api-key
```


## 2.3 Recommended Starting Configuration

For most homelabs, the defaults are a solid starting point:

- **`SEARCH_AMOUNT=5`** with **`SEARCH_INTERVAL=1hour`** will gradually cycle through your entire library
- Enable **`USE_CUTOFF=true`** if you want to skip items already at your desired quality — this avoids unnecessary API calls
- Use **`DRY_RUN=true`** first to see what Fetcharr *would* search before letting it run for real
- Set **`LOG_MODE=debug`** during initial setup to see detailed activity, then drop back to `info`



# <img src="/patreon-light.png" class="tab-icon"> 3 · Video

[![](/2026-03-24-replacing-huntarr-with-newtarr--promo-card.png)](
https://www.patreon.com/posts/replacing-with-153191561)