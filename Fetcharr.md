---
title: Fetcharr
description: A guide to deploy Fetcharr
published: true
date: 2026-03-13T15:52:59.553Z
tags: 
editor: markdown
dateCreated: 2026-03-13T15:52:59.553Z
---

# <img src="/fetcharr.png" class="tab-icon"> What is Fetcharr?

**Fetcharr** is a lightweight CLI container that periodically scans your \*arr stack (Radarr, Sonarr, Whisparr) for missing or upgradable media and automatically triggers searches. It was created as a successor to the now-defunct Huntarr project, designed to do one thing well — hunt for better quality releases so you don't have to spend hours manually searching. 

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Fetcharr

```yaml
services:
  fetcharr:
    image: egg82/fetcharr:latest
    container_name: fetcharr
    environment:
      - VERIFY_CERTS=true
      - SSL_PATH=/etc/ssl/certs/ca-bundle.crt
      - SEARCH_AMOUNT=5
      - SEARCH_INTERVAL=1hour
      - MONITORED_ONLY=true
      - USE_CUTOFF=false
      - RADARR_0_URL=https://radarr.yourdomain.com
      - RADARR_0_API_KEY=your-radarr-api-key
      - SONARR_0_URL=https://sonarr.yourdomain.com
      - SONARR_0_API_KEY=your-sonarr-api-key
    volumes:
      - /mnt/tank/configs/fetcharr:/data
    restart: unless-stopped
```

1. Replace the `RADARR_0_URL` and `SONARR_0_URL` values with the actual URLs for your *arr instances
2. Replace the `API_KEY` values with real API keys from each *arr app (found under **Settings → General**)
3. Adjust `SEARCH_AMOUNT` and `SEARCH_INTERVAL` to control how aggressively Fetcharr hunts for media


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
| `SKIP_TAGS` | *(none)* | Comma-separated list of tags to exclude from searches |
| `DRY_RUN` | `false` | Simulate searches without triggering them |
| `DATA_DIR` | `/data` | Data storage directory inside the container |
| `VERIFY_CERTS` | `true` | Verify SSL certificates when connecting to *arr apps |
| `USE_CACHE` | `true` | Enable internal caching |
| `SHORT_CACHE_TIME` | `65minutes` | TTL for short-lived cache entries |
| `LONG_CACHE_TIME` | `6hours` | TTL for long-lived cache entries |
{.dense}

## 2.2 Per-Instance Overrides

Each *arr instance can override the global `SEARCH_AMOUNT`, `SEARCH_INTERVAL`, `MONITORED_ONLY`, `SKIP_TAGS`, and `USE_CUTOFF` settings by using the instance-specific prefix. For example:

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

*Video coming soon — check back later!*