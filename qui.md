---
title: Qui
description: A guide to deploying Qui
published: true
date: 2026-01-28T15:00:30.715Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:46.966Z
---

# <img src="/autobrr.png" class="tab-icon"> What is Qui?

**Qui** is a fast, modern web interface for qBittorrent. It supports managing multiple qBittorrent instances from a single, lightweight application with features like cross-seeding, automations, orphan scanning, and backups.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Qui

```yaml
services:
  qui:
    image: ghcr.io/autobrr/qui:latest
    container_name: qui
    restart: unless-stopped
    ports:
      - "7476:7476"
    volumes:
      - /mnt/tank/configs/qui:/config
      - /mnt/tank/media:/media
```

> The media volume mount must match exactly what qBittorrent uses. This is required for orphan scan, hardlink detection, cross-seed reflinks, and automations to work.
{.is-warning}

# 2 · Initial Setup

1. Navigate to `http://{IP}:7476`
1. Set a username and password
1. Click **Add Instance** and enter your qBittorrent connection details
1. Enable **Local Filesystem Access** in the instance settings

## 2.1 Add Indexers

1. Navigate to **Settings → Indexers**
1. Click **1-click sync** to import from Prowlarr or Jackett

> **Only sync your private trackers!** Public trackers aren't useful for cross-seeding and will cause rate limit errors.
{.is-warning}

## 2.2 Add *arr Integration (Optional)

1. Navigate to **Settings → Integrations**
1. Add your Sonarr/Radarr instances

This enables IMDb/TMDb ID lookups for better cross-seed match accuracy.

# 3 · Configure Cross-Seed

Cross-seeding allows you to seed the same content on multiple trackers automatically.

## 3.1 Rules Tab

1. Navigate to **Cross-Seed → Rules**
1. Expand **Hardlink / Reflink Mode** and select your instance
1. Set **Cross-seed mode** to **Reflink** (for ZFS/Btrfs) or **Hardlink**
1. Set **Base directory** to a folder on the same filesystem as your downloads (e.g., `/media/downloads/crossseed`)
1. Set **Directory organization** to **Flat**
1. Under **Categories**, select **Category affix** with Suffix: `.cross`

> Reflink mode is safer if your filesystem supports it (ZFS, Btrfs). It creates copy-on-write clones so any writes don't affect your original files.
{.is-info}

## 3.2 Auto Tab

### RSS Automation

1. Toggle **Enable RSS automation** on
1. Set **RSS run interval** to 60-120 minutes
1. Select your qBit instance under **Target instances**
1. Add `radarr.cross` and `sonarr.cross` to **Exclude categories**
1. Click **Save RSS automation settings**

### Auto-search on completion

1. Toggle your qBit instance **On**
1. Expand the instance settings
1. Add `radarr.cross` and `sonarr.cross` to **Exclude categories**

> Excluding `.cross` categories prevents qui from trying to cross-seed torrents that are already cross-seeded.
{.is-info}

# 4 · Configure Automations

Automations are rule-based actions that automatically manage your torrents based on conditions.

## 4.1 Remove Unlinked (Upgraded Torrents)

This removes torrents that are no longer hardlinked to your media library after Radarr/Sonarr upgrades.

1. Navigate to **Automations** and click **Add rule**
1. Set **Name** to `Remove Unlinked`
1. Add condition: `Hardlink Scope` **is not** `Outside qBittorrent (library/import)`
1. Add condition: `Completed Age` **>=** `15` **days**
1. Set **Action** to **Delete** with mode **Remove with files (include cross-seeds)**
1. Leave **Include hardlinked copies** unchecked
1. Click **Create** and enable the rule

**How it works:**
- You download a movie → hardlinked to `/media` → scope = "Outside qBittorrent"
- Radarr upgrades to better quality → old hardlink deleted → scope becomes "None"
- After 15 days → rule matches → torrent and all cross-seeds deleted


## 4.2 Remove Unregistered Torrents

This removes torrents that the tracker no longer recognizes.

1. Click **Add rule**
1. Set **Name** to `Remove Unregistered`
1. Add condition: `Unregistered` **is** `true`
1. Add condition: `Completed Age` **>=** `1` **day**
1. Set **Action** to **Delete** with mode **Remove with files (include cross-seeds)**
1. Leave **Include hardlinked copies** unchecked
1. Click **Create** and enable the rule

> The 1-day grace period prevents deletion during temporary tracker issues.
{.is-info}

## 4.3 Remove Stalled Downloads (Safe)

This removes downloads that never started — safe for private trackers since you haven't crossed the Hit & Run threshold.

1. Click **Add rule**
1. Set **Name** to `Remove Stalled (No H&R Risk)`
1. Add condition: `Progress` **<** `2`
1. Add condition: `State` **is** `stalled`
1. Add condition: `Added Age` **>=** `1` **hour**
1. Set **Action** to **Delete** with mode **Remove with files**
1. Click **Create** and enable the rule

## 4.4 Tag Stalled Downloads (H&R Risk)

This tags stuck downloads that have Hit & Run risk for manual review.

1. Click **Add rule**
1. Set **Name** to `Tag Stuck (H&R Risk)`
1. Add condition: `Progress` **>=** `2`
1. Add condition: `Progress` **<** `100`
1. Add condition: `Added Age` **>=** `2` **days**
1. Set **Action** to **Tag** with tag `stuck-hr-risk`
1. Click **Create** and enable the rule

> Don't auto-delete torrents with H&R risk — you may still owe the tracker. Investigate manually or find an alternative source.
{.is-warning}

# 5 · Configure Orphan Scan

Orphan scan finds files on disk that have no corresponding torrent in qBittorrent.

1. Navigate to **Automations** and expand **Orphan Scan**
1. Toggle your instance **On**

> Orphan scan only checks directories where at least one torrent points. If you delete all torrents from a directory, leftover files there won't be detected.
{.is-info}

# 6 · Enable Reannounce

Reannounce helps fix torrents that stall right after being added — especially useful with private trackers.

1. Navigate to **Automations** and expand **Reannounce**
1. Toggle your instance **On**

# <img src="/patreon-light.png" class="tab-icon"> 7 · Video

[![](/2025-09-29-qui-a-better-qbit-interface-promo-card.png)](https://www.patreon.com/posts/qui-better-qbit-139484651)