---
title: Medialyze
description: A guide to deploying Medialyze
published: true
date: 2026-03-21T11:29:38.183Z
tags: 
editor: markdown
dateCreated: 2026-03-21T11:29:38.183Z
---

# What is MediaLyze?

**MediaLyze** is a self-hosted media library analysis tool built for large video collections. It scans your libraries using `ffprobe` and lets you explore technical metadata — formats, streams, subtitles, quality scores, and more — through a clean web UI powered by FastAPI and React. MediaLyze is read-only on your files and focuses entirely on analysis, not playback, scraping, or file modification.

Everything runs in a single container with one SQLite database and one UI. It supports full and incremental scans, normalized metadata, and detection of both internal and external subtitle files.

> 
> MediaLyze mounts your media directory as **read-only**. It will never modify, move, or delete your files.
{.is-success}

# <img src="/docker.png" class="tab-icon"> 1 · Deploy MediaLyze

```yaml
services:
  medialyze:
    image: ghcr.io/frederikemmer/medialyze:latest
    container_name: medialyze
    environment:
      - TZ=America/New_York
    restart: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - /mnt/tank/configs/medialyze:/config
      - /mnt/tank/media:/media:ro
```

# 2 · Configuration


## 2.1 Initial Setup

Once the container is running:

1. Open `http://your-server-ip:8080` in your browser
2. Navigate to **Settings** to add your media library paths
3. Trigger a **full scan** to index your collection
4. Explore the **Dashboard** for an overview of your library's technical metadata

> 
> MediaLyze uses `path + size + mtime` for incremental scans, so subsequent scans after the initial full scan will be significantly faster.
{.is-info}
