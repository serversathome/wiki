---
title: Houndarr
description: A guide to deploy Houndarr
published: true
date: 2026-03-27T14:32:54.955Z
tags: 
editor: markdown
dateCreated: 2026-03-27T14:32:54.954Z
---

# <img src="/houndarr.png" class="tab-icon"> What is Houndarr?

**Houndarr** is a polite, automated media search tool for your \*arr stack. Radarr, Sonarr, Lidarr, Readarr, and Whisparr monitor RSS feeds for new releases, but they don't actively go back and search for content that's missing or below your quality cutoff. Their built-in "Search All Missing" button fires every request at once, which can overwhelm your indexers and get you banned.

Houndarr solves this by searching **slowly and automatically** — small batches, configurable sleep intervals, per-item cooldowns, and hourly API caps. It works with **Radarr**, **Sonarr**, **Lidarr**, **Readarr**, and **Whisparr**, supporting multiple instances of each. It runs as a single Docker container with a dark-themed web UI, SQLite database, encrypted API keys, and zero telemetry.



# <img src="/docker.png" class="tab-icon"> 1 · Deploy Houndarr

```yaml
services:
  houndarr:
    image: ghcr.io/av1155/houndarr:latest
    container_name: houndarr
    restart: unless-stopped
    ports:
      - "8877:8877"
    volumes:
      - /mnt/tank/configs/houndarr:/data
    environment:
      - TZ=America/New_York
      - PUID=568
      - PGID=568
```


# 2 · Configuration

## 2.1 Adding \*arr Instances

1. Log in and navigate to **Settings**
2. Click **Add Instance**
3. Select the instance type: Radarr, Sonarr, Lidarr, Readarr, or Whisparr
4. Enter the **URL** of your \*arr instance (e.g. `http://192.168.1.100:7878`)
5. Enter the **API Key** (found in your \*arr instance under Settings → General)
6. Configure search parameters (see below)
7. Toggle the instance **Enabled** and save

> 
> API keys are encrypted at rest using Fernet (AES-128-CBC + HMAC-SHA256) and are never sent back to the browser.
{.is-success}

## 2.2 Instance Search Settings

Each instance has independent controls for missing and cutoff searches:

**Missing Search Controls:**

| Setting | Default | Description |
|---|---|---|
| Batch Size | `2` | Max missing items considered per cycle |
| Sleep (minutes) | `30` | Wait time between cycles |
| Hourly Cap | `4` | Max successful missing searches per hour |
| Cooldown (days) | `14` | Min days before retrying the same item |
| Post-Release Grace (hours) | `6` | Hours to wait after release date before searching |
| Queue Limit | `0` | Skip cycle if download queue meets/exceeds this (0 = disabled) |


**Cutoff Upgrade Controls:**

| Setting | Default | Description |
|---|---|---|
| Cutoff Search | Off | Enable searching for items below quality cutoff |
| Cutoff Batch | `1` | Max cutoff items per cycle |
| Cutoff Cooldown (days) | `21` | Min days before retrying the same cutoff item |
| Cutoff Cap | `1` | Max successful cutoff searches per hour |


> 
> Start with the defaults and increase one control at a time. Watch your logs for a full day before bumping further. Suggested order: Batch Size → Sleep → Hourly Cap → Cutoff Search (last).
{.is-info}

# 3 · How It Works

Houndarr runs a continuous loop for each enabled instance:

1. **Fetch wanted list** — Queries your \*arr instance for missing items (and optionally cutoff-unmet items)
2. **Filter candidates** — Skips items that are on cooldown, not yet released, or within the post-release grace window
3. **Send search commands** — Triggers searches in small batches through the \*arr API
4. **Sleep** — Waits the configured interval before the next cycle
5. **Repeat**

Houndarr uses **fair backlog scanning**, meaning it doesn't stop at the first page of results. It scans deeper pages when top candidates are ineligible, but stays bounded with hard caps on page walks and scan budgets.

**What Houndarr does NOT do:**

- No download client integration — your \*arr instances handle downloads
- No Prowlarr/indexer management — your \*arr instances manage their own indexers
- No request workflows (no Overseerr/Ombi-style features)
- No multi-user support — single admin account only
- No media file manipulation — it never touches your library files

