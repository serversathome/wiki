---
title: qBit Manager
description: A guide to deploying qBit Manage
published: true
date: 2025-09-11T09:49:37.430Z
tags: 
editor: markdown
dateCreated: 2025-09-11T09:40:32.816Z
---

> UNDER CONSTRUCTION
{.is-danger}



# ![](/qbit-manage.png){class="tab-icon"} What is qBit Manage?
This is a program used to manage your qBittorrent instance such as:

- Tag torrents based on tracker URLs
- Apply category based on save_path to uncategorized torrents in category's save_path
- Change categories based on current category (cat_change)
- Remove unregistered torrents (delete data & torrent if it is not being cross-seeded, otherwise it will just remove the torrent)
- Recheck paused torrents sorted by lowest size and resume if completed
- Remove orphaned files from your root directory that are not referenced by qBittorrent
- Tag any torrents that have no hard links outside the root folder (for multi-file torrents the largest file is used)
- Apply share limits based on groups filtered by tags/categories and allows optional cleanup to delete these torrents and contents based on maximum ratio and/or time seeded. Additionally allows for a minimum seed time to ensure tracker rules are respected and minimum number of seeders to keep torrents alive.
- RecycleBin function to move files into a RecycleBin folder instead of deleting the data directly when deleting a torrent
- Built-in scheduler to run the script every x minutes. (Can use --run command to run without the scheduler)
- Webhook notifications with Notifiarr and Apprise API integration


# <img src="/docker.png" class="tab-icon"> 1 · Deploy qBit Manage
```yaml
services:
  qbit_manage:
    container_name: qbit_manage
    image: ghcr.io/stuffanthings/qbit_manage:latest
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/qbitmanage/:/config:rw
      - /mnt/tank/media/downloads/:/media/downloads:rw
      - /mnt/tank/configs/qbittorrent/:/qbittorrent/:ro
    ports:
      - "8081:8080"  # Web API port (when enabled)
    environment:
      - QBT_WEB_SERVER=true
      - QBT_PORT=8080

      # Scheduler Configuration
      - QBT_RUN=false
      - QBT_SCHEDULE=1440
      - QBT_CONFIG_DIR=/config
      - QBT_LOGFILE=qbit_manage.log

      # Command Flags
      - QBT_RECHECK=false
      - QBT_CAT_UPDATE=false
      - QBT_TAG_UPDATE=false
      - QBT_REM_UNREGISTERED=false
      - QBT_REM_ORPHANED=false
      - QBT_TAG_TRACKER_ERROR=false
      - QBT_TAG_NOHARDLINKS=false
      - QBT_SHARE_LIMITS=false
      - QBT_SKIP_CLEANUP=false
      - QBT_DRY_RUN=false
      - QBT_STARTUP_DELAY=0
      - QBT_SKIP_QB_VERSION_CHECK=false
      - QBT_DEBUG=false
      - QBT_TRACE=false

      # Logging Configuration
      - QBT_LOG_LEVEL=INFO
      - QBT_LOG_SIZE=10
      - QBT_LOG_COUNT=5
      - QBT_DIVIDER==
      - QBT_WIDTH=100
```


# 2 · 