---
title: Qui
description: A guide to deploying Qui
published: true
date: 2026-01-28T14:28:07.661Z
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


# 2 · Initial Setup

1. Navigate to `http://{IP}:7476`
2. Set a username and password
3. Click **Add Instance** and enter your qBittorrent connection details
4. Enable **Local Filesystem Access** in the instance settings

## 2.1 Add Indexers

1. Navigate to **Settings → Indexers**
2. Click **1-click sync** to import from Prowlarr or Jackett

> Only sync your private trackers. Public trackers like The Pirate Bay aren't useful for cross-seeding and will cause rate limit errors.
{.is-info}

## 2.2 Add *arr Integration (Optional)

1. Navigate to **Settings → Integrations**
2. Add your Sonarr/Radarr instances

This enables IMDb/TMDb ID lookups for better cross-seed match accuracy.

# 3 · Configure Cross-Seed

Cross-seeding allows you to seed the same content on multiple trackers automatically.

## 3.1 Rules Tab

1. Navigate to **Cross-Seed → Rules**
2. Expand **Hardlink / Reflink Mode** and select your instance
3. Set **Cross-seed mode** to **Reflink** (for ZFS/Btrfs) or **Hardlink**
4. Set **Base directory** to a folder on the same filesystem as your downloads (e.g., `/media/downloads/crossseed`)
5. Set **Directory organization** to **Flat**
6. Under **Categories**, select **Category affix** with Suffix: `.cross`

> Reflink mode is safer if your filesystem supports it (ZFS, Btrfs). It creates copy-on-write clones so any writes don't affect your original files.
{.is-info}

## 3.2 Auto Tab

### RSS Automation

1. Toggle **Enable RSS automation** on
2. Set **RSS run interval** to 60-120 minutes
3. Select your qBit instance under **Target instances**
4. Add `radarr.cross` and `sonarr.cross` to **Exclude categories**
5. Click **Save RSS automation settings**

### Auto-search on completion

1. Toggle your qBit instance **On**
2. Expand the instance settings
3. Add `radarr.cross` and `sonarr.cross` to **Exclude categories**

> Excluding `.cross` categories prevents qui from trying to cross-seed torrents that are already cross-seeded.
{.is-info}

# 4 · Configure Automations

Automations are rule-based actions that automatically manage your torrents based on conditions.

## 4.1 Remove Unlinked (Upgraded Torrents)

This removes torrents that are no longer hardlinked to your media library after Radarr/Sonarr upgrades.

1. Navigate to **Automations** and click **Add rule**
2. Set **Name** to `Remove Unlinked`
3. Add condition: `Hardlink scope` **is not** `Outside qBittorrent (library/import)`
4. Add condition: `Completed Age` **>=** `15` **days**
5. Set **Action** to **Delete** with mode **Remove with files (include cross-seeds)**
6. Leave **Include hardlinked copies** unchecked
7. Click **Create** and enable the rule

**How it works:**
- You download a movie → hardlinked to `/media` → scope = "Outside qBittorrent"
- Radarr upgrades to better quality → old hardlink deleted → scope becomes "None"
- After 15 days → rule matches → torrent and all cross-seeds deleted

> Leave "Include hardlinked copies" unchecked or it will delete your media library copy!
{.is-warning}

## 4.2 Remove Unregistered Torrents

This removes torrents that the tracker no longer recognizes.

1. Click **Add rule**
2. Set **Name** to `Remove Unregistered`
3. Add condition: `Is Unregistered` **is** `true`
4. Add condition: `Completed Age` **>=** `1` **day**
5. Set **Action** to **Delete** with mode **Remove with files (include cross-seeds)**
6. Leave **Include hardlinked copies** unchecked
7. Click **Create** and enable the rule

> The 1-day grace period prevents deletion during temporary tracker issues.
{.is-info}

## 4.3 Remove Stalled Downloads (Safe)

This removes downloads that never started — safe for private trackers since you haven't crossed the Hit & Run threshold.

1. Click **Add rule**
2. Set **Name** to `Remove Stalled (No H&R Risk)`
3. Add condition: `Progress` **<** `0.02`
4. Add condition: `State` **is** `stalled`
5. Add condition: `Added On Age` **>=** `1` **hour**
6. Set **Action** to **Delete** with mode **Remove with files**
7. Click **Create** and enable the rule

## 4.4 Tag Stalled Downloads (H&R Risk)

This tags stuck downloads that have Hit & Run risk for manual review.

1. Click **Add rule**
2. Set **Name** to `Tag Stuck (H&R Risk)`
3. Add condition: `Progress` **>=** `0.02`
4. Add condition: `Progress` **<** `1`
5. Add condition: `Last Activity Age` **>=** `3` **days**
6. Set **Action** to **Tag** with tag `stuck-hr-risk`
7. Click **Create** and enable the rule

> Don't auto-delete torrents with H&R risk — you may still owe the tracker. Investigate manually or find an alternative source.
{.is-warning}

# 5 · Configure Orphan Scan

Orphan scan finds files on disk that have no corresponding torrent in qBittorrent.

1. Navigate to **Automations** and expand **Orphan Scan**
2. Toggle your instance **On**
3. Click the settings icon and add any paths to ignore (e.g., `/media/downloads/VueTorrent`)

> Orphan scan only checks directories where at least one torrent points. If you delete all torrents from a directory, leftover files there won't be detected.
{.is-info}

# 6 · Enable Reannounce

Reannounce helps fix torrents that stall right after being added — especially useful with private trackers.

1. Navigate to **Automations** and expand **Reannounce**
2. Toggle your instance **On**

# <img src="/patreon-light.png" class="tab-icon"> 8 · Video

[![](/2025-09-29-qui-a-better-qbit-interface-promo-card.png)](https://www.patreon.com/posts/qui-better-qbit-139484651)