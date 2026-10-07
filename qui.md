---
title: Qui
description: A guide to deploying Qui
published: true
date: 2026-10-07T16:12:44.974Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:46.966Z
---

# <img src="/autobrr.png" class="tab-icon"> What is Qui?

**Qui** is a fast, modern web interface for qBittorrent. It supports managing multiple qBittorrent instances from a single, lightweight application with features like cross-seeding, automations, orphan scanning, and backups.

<div class="glance">
  <div><span>Port</span><b><code>7476</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty intermediate">Intermediate</b></div>
  <div class="glance-links">
    <a href="https://github.com/autobrr/qui"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Qui

```yaml
services:
  qui:
    image: ghcr.io/autobrr/qui:latest
    container_name: qui
    user: "568:568"
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

## 2.2 Add \*arr Integration (Optional)

1. Navigate to **Settings → Integrations**
1. Add your Sonarr/Radarr instances

This enables IMDb/TMDb ID lookups for better cross-seed match accuracy.

# 3 · Configure Cross-Seed

Cross-seeding allows you to seed the same content on multiple trackers automatically. Qui finds cross-seeds three ways, and you want all three:

- **Library Scan** searches your trackers for everything you already seed. This is how your existing library gets cross-seeded.
- **RSS Automation** watches your trackers' feeds for new uploads that match something you already have.
- **Auto-search on completion** searches for cross-seeds as soon as a new download finishes.

## 3.1 Rules Tab

1. Navigate to **Cross-Seed → Rules**
1. Expand **Hardlink / Reflink Mode** and select your instance
1. Set **Cross-seed mode** to **Reflink** (for ZFS/Btrfs) or **Hardlink**
1. Set **Base directory** to a folder on the same filesystem as your downloads (e.g., `/media/downloads/crossseed`)
1. Set **Directory organization** to **Flat**
1. Under **Categories**, select **Category affix** with Suffix: `.cross`
1. Set **Size Mismatch Tolerance** to `0` (or `0.1%`)
1. Toggle **Piece Boundary Safety Check** on

> Reflink mode is safer if your filesystem supports it (ZFS, Btrfs). It creates copy-on-write clones so any writes don't affect your original files.
{.is-info}

> The piece boundary check matters most in **Hardlink** mode. It blocks cross-seeds whose extra files share pieces with your content, which could otherwise let qBittorrent overwrite your existing data.
{.is-warning}

## 3.2 Library Scan (Existing Library)

RSS only sees new uploads, so it will never find matches for content you already have. Run a Library Scan once to backfill your existing library.

1. Navigate to **Cross-Seed → Scan**
1. Select your qBit instance
1. Under **Categories**, select your \*arr categories (e.g. `radarr` and `sonarr`, or `movies` and `tv`)
1. Set **Interval** to `60` seconds
1. Set **Cooldown** to `7` days
1. Click **Start**

> Library Scan queries every indexer for every torrent. At 60 seconds per torrent, 1,000 torrents takes about 17 hours. Run it once, then rerun it occasionally to catch older content that shows up on new trackers. RSS handles everything new.
{.is-warning}

> Only select your original categories here. Your `.cross` categories hold cross-seeds of the same content, so scanning them just repeats the work.
{.is-info}

## 3.3 RSS Automation

1. Navigate to **Cross-Seed → Auto**
1. Toggle **Enable RSS automation** on
1. Set **RSS run interval** to 60-120 minutes
1. Select your qBit instance under **Target instances**
1. Leave **Target indexers** empty to use all enabled indexers
1. Add `cross-seed` to **Exclude tags**
1. Click **Save RSS automation settings**

> RSS compares new uploads in your trackers' feeds against torrents you already have. Excluding the `cross-seed` tag makes qui match against your original torrents instead of existing cross-seeds. Qui tags every cross-seed with `cross-seed` by default, so this works no matter what your categories are called.
{.is-info}

## 3.4 Auto-search on Completion

1. Toggle your qBit instance **On**
1. Expand the instance settings
1. Add `cross-seed` to **Exclude tags**

> New cross-seeds finish a recheck after they're added, which counts as a completion. Excluding the `cross-seed` tag stops every cross-seed from triggering another search.
{.is-info}

# 4 · Configure Automations

Automations are rule-based actions that automatically manage your torrents based on conditions. To use these, Click the **Import** button and paste these contents in.

## 4.1 Remove Unlinked (Upgraded Torrents)

This removes torrents that are no longer hardlinked to your media library after Radarr/Sonarr upgrades.

> **Read before enabling.** This rule deletes anything that isn't hardlinked into your library. If your downloads and media folders are on different datasets, your \*arr apps copied files instead of hardlinking them, and **every torrent you have** will match. Torrents your \*arr apps never imported (music, ISOs, manual downloads) will match too. Limit the rule to your \*arr categories, and test it with a **Tag** action before switching it to **Delete**.
{.is-danger}

```json
{
  "name": "Remove Unlinked",
  "trackerPattern": "*",
  "trackerDomains": [
    "*"
  ],
  "conditions": {
    "schemaVersion": "1",
    "delete": {
      "enabled": true,
      "mode": "deleteWithFilesIncludeCrossSeeds",
      "includeHardlinks": true,
      "condition": {
        "operator": "AND",
        "conditions": [
          {
            "field": "HARDLINK_SCOPE",
            "operator": "NOT_EQUAL",
            "value": "outside_qbittorrent"
          },
          {
            "field": "COMPLETION_ON_AGE",
            "operator": "GREATER_THAN_OR_EQUAL",
            "value": "1296000"
          }
        ]
      }
    }
  }
}
```

**How it works:**
- You download a movie → hardlinked to `/media` → scope = "Outside qBittorrent"
- Radarr upgrades to better quality → old hardlink deleted → scope becomes "None"
- After 15 days → rule matches → torrent and all cross-seeds deleted

> Some trackers require more than 15 days of seeding. Raise **Completed Age** to match the longest seeding requirement across your trackers.
{.is-warning}

## 4.2 Remove Unregistered Torrents

This removes torrents that the tracker no longer recognizes.

```json
{
  "name": "Unregistered",
  "trackerPattern": "*",
  "trackerDomains": [
    "*"
  ],
  "conditions": {
    "schemaVersion": "1",
    "delete": {
      "enabled": true,
      "mode": "deleteWithFilesIncludeCrossSeeds",
      "condition": {
        "operator": "AND",
        "conditions": [
          {
            "field": "IS_UNREGISTERED",
            "operator": "EQUAL",
            "value": "true"
          },
          {
            "field": "COMPLETION_ON_AGE",
            "operator": "GREATER_THAN_OR_EQUAL",
            "value": "86400"
          }
        ]
      }
    }
  }
}
```

> The 1-day grace period prevents deletion during temporary tracker issues.
{.is-info}

## 4.3 Remove Stalled Downloads (Safe)

This removes downloads that never started. It's safe for private trackers because you haven't crossed the Hit & Run threshold.

```json
{
  "name": "Stalled",
  "trackerPattern": "*",
  "trackerDomains": [
    "*"
  ],
  "conditions": {
    "schemaVersion": "1",
    "delete": {
      "enabled": true,
      "mode": "deleteWithFiles",
      "condition": {
        "operator": "AND",
        "conditions": [
          {
            "field": "PROGRESS",
            "operator": "LESS_THAN_OR_EQUAL",
            "value": "0.02"
          },
          {
            "field": "ADDED_ON_AGE",
            "operator": "GREATER_THAN_OR_EQUAL",
            "value": "3600"
          }
        ]
      }
    }
  }
}
```

## 4.4 Tag Stalled Downloads (H&R Risk)

This tags stuck downloads that have Hit & Run risk for manual review.

```json
{
  "name": "Tag Stalled (H&R Risk)",
  "trackerPattern": "*",
  "trackerDomains": [
    "*"
  ],
  "conditions": {
    "schemaVersion": "1",
    "tag": {
      "enabled": true,
      "tags": [
        "stuck-hr-risk"
      ],
      "mode": "full",
      "condition": {
        "operator": "AND",
        "conditions": [
          {
            "field": "PROGRESS",
            "operator": "GREATER_THAN_OR_EQUAL",
            "value": "0.02"
          },
          {
            "field": "PROGRESS",
            "operator": "LESS_THAN",
            "value": "1"
          },
          {
            "field": "ADDED_ON_AGE",
            "operator": "GREATER_THAN_OR_EQUAL",
            "value": "172800"
          }
        ]
      }
    },
    "tags": [
      {
        "enabled": true,
        "tags": [
          "stuck-hr-risk"
        ],
        "mode": "full",
        "condition": {
          "operator": "AND",
          "conditions": [
            {
              "field": "PROGRESS",
              "operator": "GREATER_THAN_OR_EQUAL",
              "value": "0.02"
            },
            {
              "field": "PROGRESS",
              "operator": "LESS_THAN",
              "value": "1"
            },
            {
              "field": "ADDED_ON_AGE",
              "operator": "GREATER_THAN_OR_EQUAL",
              "value": "172800"
            }
          ]
        }
      }
    ]
  }
}
```

> Don't auto-delete torrents with H&R risk. You may still have unmet seeding requirements. Investigate manually or find an alternative source.
{.is-warning}

# 5 · Configure Orphan Scan

Orphan scan finds files on disk that have no corresponding torrent in qBittorrent.

1. Navigate to **Automations** and expand **Orphan Scan**
1. Toggle your instance **On**

> Orphan scan only checks directories where at least one torrent points. If you delete all torrents from a directory, leftover files there won't be detected.
{.is-info}

# 6 · Enable Reannounce

Reannounce helps fix torrents that stall right after being added. It's especially useful with private trackers.

1. Navigate to **Automations** and expand **Reannounce**
1. Toggle your instance **On**

# <img src="/patreon-light.png" class="tab-icon"> 7 · Video

[![](/2025-09-29-qui-a-better-qbit-interface-promo-card.png)](https://www.patreon.com/posts/qui-better-qbit-139484651)