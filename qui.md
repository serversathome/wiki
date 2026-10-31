---
title: Qui
description: A guide to deploying Qui
published: true
date: 2026-10-19T14:28:07.465Z
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
1. Click **Discover**, enter your Prowlarr or Jackett URL and API key, then select the indexers to import

> **Only sync your private trackers!** Public trackers aren't useful for cross-seeding and will cause rate limit errors.
{.is-warning}

## 2.2 Add \*arr Integration (Optional)

1. Navigate to **Settings → Integrations**
1. Add your Sonarr/Radarr instances

This enables IMDb/TMDb ID lookups for better cross-seed match accuracy.

# 3 · Configure Cross-Seed

Cross-seeding allows you to seed the same content on multiple trackers automatically. Everything lives on the **Cross-Seed** page, which has a sidebar split into three groups:

- **Sources**: how qui finds cross-seeds (**RSS**, **Webhook**, **Completion**, **Library**, **Directories**)
- **Matching**: **Season packs** and **Matching rules**
- **Injection**: **Categories and tags** and **After injection**

Set up the global settings (3.1 to 3.3) first, then turn on the sources you want (3.4 onward). Each card has its own **Save Changes** button, so save before moving to the next tab.

## 3.1 Hardlink Mode

1. Navigate to **Cross-Seed → After injection**
1. Expand **Hardlink / Reflink Mode** and expand your instance
1. Set **Cross-seed mode** to **Hardlink**
1. Set **Base directories** to a folder on the same filesystem as your downloads (e.g., `/media/downloads/crossseed`)
1. Set **Directory organization** to **Flat (all in base directory)**
1. Click **Save Changes**

> Reflink mode is safer if your filesystem supports it (Btrfs, or XFS with `reflink=1`). Check with `df -T /path`. It creates copy-on-write clones so any writes don't affect your original files.
{.is-info}

> Hardlink and reflink modes need **Local Filesystem Access** turned on for the instance (step 2).
{.is-warning}

## 3.2 Matching Rules

1. Navigate to **Cross-Seed → Matching rules**
1. Under **Safety & validation**, toggle the **Piece boundary safety check** on
1. Click **Save Changes**

> The piece boundary check is **off by default**. It matters most in **Hardlink** mode. It blocks cross-seeds whose extra files share pieces with your content, which could otherwise let qBittorrent overwrite your existing data.
{.is-warning}

## 3.3 Categories

1. Navigate to **Cross-Seed → Categories and tags**
1. Select **Category affix**, choose **Suffix**, and enter `.cross`
1. Click **Save Changes**

> Every source tags its cross-seeds with `cross-seed` by default. You can change the tags on each source's tab, but leave `cross-seed` in place. The steps below rely on it.
{.is-info}

## 3.4 RSS Automation

RSS watches your trackers' feeds for new uploads that match something you already have.

1. Navigate to **Cross-Seed → RSS**
1. Toggle **Enable RSS automation** on
1. Set **RSS run interval (minutes)** to `60`-`120`
1. Select your qBit instance under **Target instances**
1. Leave **Target indexers** empty to use all enabled indexers
1. Add `cross-seed` to **Exclude tags**
1. Click **Save RSS automation settings**

> Excluding the `cross-seed` tag makes qui match against your original torrents instead of existing cross-seeds.
{.is-info}

## 3.5 Auto-search on Completion

Completion search looks for cross-seeds as soon as a new download finishes.

1. Navigate to **Cross-Seed → Completion**
1. In the **Auto-search on completion** card, toggle your qBit instance **On**
1. Expand the instance and add `cross-seed` to the **Exclude filters** tags
1. Save the instance settings

> New cross-seeds finish a recheck after they're added, which counts as a completion. Excluding the `cross-seed` tag stops every cross-seed from triggering another search.
{.is-info}

## 3.6 Library Scan (Existing Library)

RSS only sees new uploads, so it will never find matches for content you already have. Run a Library Scan once to backfill your existing library.

1. Navigate to **Cross-Seed → Library**
1. Select your qBit instance as the **Source instance**
1. Under **Categories**, select your \*arr categories (e.g. `radarr` and `sonarr`, or `movies` and `tv`)
1. Set **Interval between torrents (seconds)** to `60`
1. Set **Cooldown (minutes)** to `10080` (7 days)
1. Click **Start run**

> Library Scan queries every indexer for every torrent. At 60 seconds per torrent, 1,000 torrents takes about 17 hours. Run it once, then rerun it occasionally to catch older content that shows up on new trackers. RSS handles everything new.
{.is-warning}

> Only select your original categories here. Your `.cross` categories hold cross-seeds of the same content, so scanning them just repeats the work.
{.is-info}

## 3.7 Dir Scan (Optional)

Library Scan only looks at torrents in qBittorrent. Dir Scan looks at files on disk instead, so it can cross-seed content you removed from qBittorrent or never downloaded by torrent. It needs **Local Filesystem Access** on the target instance and read access to the folder.

### Global Settings

1. Navigate to **Cross-Seed → Directories**
1. Toggle the **Directory Scanner** switch to **Enabled**
1. Click **Settings** to open **Directory Scanner Settings**
1. Set **Match Mode** to **Strict (name + size)** (use **Flexible** only if your files have been renamed)
1. Set **Size Tolerance (%)** to `0`
1. Leave **Allow partial matches** off
1. Leave **Download missing files** on (the default)
1. Leave **Skip piece boundary safety check** off
1. Leave **Start torrents paused** off
1. Set **Max searchees per run** to `0` (unlimited), or a smaller number to spread a large first scan across several runs
1. Set **Only process items changed within the last (days)** to `0` to scan everything, or a number of days to skip old content
1. Set **Default Category** to a category for these cross-seeds (e.g. `dirscan.cross`)
1. Set **Tags** to `cross-seed`
1. Click **Save**

> Dir Scan adds torrents with an explicit save path and AutoTMM off. The **Category affix** and the source tags from the other tabs don't apply here, so the category and tags come only from these settings. Adding the `cross-seed` tag keeps the RSS and completion exclude rules working.
{.is-info}

> Leave partial matches off unless you need them. With partial matches on, qBittorrent can download missing files (like `.nfo` or subtitles) into the scanned folder.
{.is-warning}

### Add a Directory

1. Click **Add Directory** and set **Directory Path** to a single category folder (e.g. `/media/downloads/movies`)
1. Leave **qBittorrent Path Prefix** empty. The compose above mounts `/media` the same way as qBittorrent
1. Select your qBit instance as the **Target qBittorrent Instance**
1. Set **Scan Interval (minutes)** to `1440` (24 hours, the default)
1. For TV folders, toggle **Skip individual episodes** on to search for season packs
1. Click **Add Directory** to save it
1. Repeat for each category folder, then click the ▶ play button on a directory card to start a scan

> Set up hardlink or reflink mode (3.1) first. In regular mode, Dir Scan warns that the orphan scanner will treat the folder as a scan root and flag any unmatched files as orphans.
{.is-warning}

> Each item directly inside the path is searched as one release. Add one entry per category folder (movies, TV, music). Don't point it at a parent folder that holds several categories, or at a single release folder.
{.is-info}

> Dir Scan waits 60 seconds between searches, so the first scan of a large folder can take hours or days. Scanning your downloads folder is safer than scanning your \*arr library folders.
{.is-warning}

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