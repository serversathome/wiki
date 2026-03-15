---
title: Sortarr
description: A guide to deploying Sortarr
published: true
date: 2026-03-15T20:51:24.672Z
tags: 
editor: markdown
dateCreated: 2026-01-17T15:46:01.206Z
---

# What is Sortarr?

**Sortarr** is a lightweight, read-only web dashboard for Sonarr and Radarr that provides deep storage insights into your media library. It connects to your Sonarr/Radarr APIs (and optionally Tautulli) to compute size and efficiency metrics, helping you spot oversized series or movies and compare quality vs. size trade-offs. Sortarr does **not** modify files or take any actions against your media — it's purely analytical.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Sortarr

```yaml
services:
  sortarr:
    image: ghcr.io/jaredharper1/sortarr:latest
    container_name: sortarr
    ports:
      - "9595:8787"
    volumes:
      - /mnt/tank/configs/sortarr:/config
    restart: unless-stopped
```

> On first visit, you will be redirected to `/setup` where you can enter your Sonarr/Radarr URLs, API keys, and optionally configure Tautulli and basic auth.
{.is-info}


> To use Docker Hub instead of GHCR, change the image to `docker.io/jaredharper1/sortarr:latest`.
{.is-info}

# 2 · Configuration

## 2.1 Initial Setup

When you first open Sortarr, the `/setup` page will prompt you to enter your service details:

| Field | Description |
|---|---|
| **Sonarr URL** | Your Sonarr instance URL (e.g. `http://sonarr:8989`) |
| **Sonarr API Key** | Found in Sonarr under **Settings → General → API Key** |
| **Radarr URL** | Your Radarr instance URL (e.g. `http://radarr:7878`) |
| **Radarr API Key** | Found in Radarr under **Settings → General → API Key** |


URLs can be entered with or without a scheme — duplicate schemes are automatically normalized. Read-only API keys are recommended.


> Treat the `Sortarr.env` file as a secret — it stores API keys and optional basic auth credentials.
{.is-warning}

## 2.2 Multiple Instances

Sortarr supports up to 3 Sonarr and 3 Radarr instances. Additional instances are configured under the **Advanced** sections of the setup page. When multiple instances are configured, an **Instance** column and filter chips appear in the UI.

## 2.3 Tautulli Integration (Optional)

To enable playback stats, add your Tautulli URL and API key in the setup page. When configured, the following columns become available: play count, last watched, watch time, watch vs. content hours, and users.

| Variable | Default | Description |
|---|---|---|
| `TAUTULLI_URL` | — | Your Tautulli instance URL |
| `TAUTULLI_API_KEY` | — | Your Tautulli API key |
| `TAUTULLI_METADATA_WORKERS` | `4` | Parallel metadata lookup workers |
| `TAUTULLI_METADATA_LOOKUP_LIMIT` | `-1` | Max lookups (`-1` = no limit, `0` = disable) |



> The first load after startup can take a while for large libraries, especially with Tautulli enabled. Later loads use the persistent cache and are much faster.
{.is-info}

## 2.4 Basic Auth (Optional)

For deployments exposed beyond your LAN, Sortarr supports basic authentication:

| Variable | Description |
|---|---|
| `BASIC_AUTH_USER` | Username for basic auth |
| `BASIC_AUTH_PASS` | Password for basic auth |


## 2.5 Environment Variables

| Variable | Default | Description |
|---|---|---|
| `PUID` | — | Container user ID |
| `PGID` | — | Container group ID |
| `PORT` | `8787` | Internal port (Docker maps `9595` → `8787`) |
| `SORTARR_LOG_LEVEL` | `INFO` | Log verbosity |
| `SONARR_TIMEOUT_SECONDS` | `90` | Per-request timeout for Sonarr API calls |
| `RADARR_TIMEOUT_SECONDS` | `90` | Per-request timeout for Radarr API calls |
| `TAUTULLI_TIMEOUT_SECONDS` | `60` | Per-request timeout for Tautulli API calls |


> 
> When `PUID`/`PGID` are set, the container runs as that user and will chown the config/cache paths on startup.
{.is-info}

# 3 · Using Sortarr

## 3.1 Status Bar Actions

The status bar provides several refresh options:

| Button | Action |
|---|---|
| **Refresh Sonarr data** | Reloads all Sonarr series data |
| **Refresh Radarr data** | Reloads all Radarr movie data |
| **Refresh Tautulli matches** | Rebuilds Tautulli matching using existing data |
| **Refresh Tautulli media + matches** | Refreshes Tautulli library info then rebuilds matches |
| **Clear cached data** | Clears all caches and reloads from Sonarr/Radarr |


## 3.2 Advanced Filtering

Use `field:value` syntax in the search bar for powerful filtering. Numeric fields treat `field:value` as `>=` by default (use `=` for exact matches).

**Examples:**
- `audio:Atmos` — filter by Atmos audio
- `videocodec:x265` — filter by x265 codec
- `gbperhour:1` — movies using 1+ GiB per hour
- `totalsize:10` — items 10+ GiB total
- `neverwatched:true` — items never played (requires Tautulli)
- `playcount>=5` — items played 5 or more times
- `dayssincewatched>=365` — items not watched in over a year
- `resolution:4k` — filter by 4K resolution (aliases: `4k`, `uhd`, `fhd`, `hd`, `sd`)

## 3.3 CSV Export

Both Sonarr and Radarr views support CSV export. Tautulli playback columns are included only when Tautulli is configured.



# <img src="/patreon-light.png" class="tab-icon"> 5 · Video
[![](/2026-01-27-better-radarrsonarr-ui-with-sor-promo-card.png)](https://www.patreon.com/posts/better-radarr-ui-148458595)