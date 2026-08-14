---
title: Chaptarr
description: A guide to deploying Chaptarr
published: true
date: 2026-08-14T11:14:36.717Z
tags: 
editor: markdown
dateCreated: 2026-08-14T11:11:39.002Z
---

# What is Chaptarr?

**Chaptarr** is a book collection manager for audiobooks and ebooks — a community fork of the retired Readarr, rebuilt around its own metadata pipeline. You follow authors, it watches your indexers, grabs releases through your download client, renames them into a clean tree, and upgrades quality when something better appears.

It handles both media types in a single instance, which Readarr never could, and it understands narrator and edition data that general-purpose media managers ignore.

> Readarr was archived on **27 June 2025** when its metadata backend went permanently offline. Chaptarr is not compatible with Readarr's metadata sources — it resolves entities across multiple providers and aggregates them by consensus, specifically so one dead endpoint cannot kill the project again.
{.is-info}



# <img src="/docker.png" class="tab-icon"> 1 · Deploy Chaptarr

Docker is the only supported deployment method. There are no native install packages — earlier zip attachments on GitHub were incomplete build artifacts and have been withdrawn.

```yaml
services:
  chaptarr:
    image: chaptarr/chaptarr:latest
    container_name: chaptarr
    restart: unless-stopped
    ports:
      - "8789:8789"
    environment:
      - PUID=568
      - PGID=568
      - UMASK=002
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/chaptarr:/config
      - /mnt/tank/media:/media
```

Create the config directory with correct ownership first, or Docker creates it as `root:root`:

```bash
mkdir -p /mnt/tank/configs/chaptarr
mkdir -p /mnt/tank/media/books/{audiobooks,ebooks}
chown -R 568:568 /mnt/tank/configs/chaptarr /mnt/tank/media/books
```

Browse to `http://YOUR-SERVER-IP:8789` and set the username and password on first launch.


# 2 · Configuration

## 2.1 Root folders

**Settings** → **Media Management** → **Root Folders**:

| Library | Path |
|---------|------|
| Audiobooks | `/media/books/audiobooks` |
| Ebooks | `/media/books/ebooks` |


You can keep them separate or place ebooks alongside their audiobook counterparts — Chaptarr supports both layouts.


## 2.2 Indexers

Chaptarr is not a named application in Prowlarr. Add it there as a **Readarr**-type app — the API is compatible and syncing works out of the box.

1. In Prowlarr: **Settings** → **Apps** → **+** → **Readarr**
2. **Prowlarr Server**: `http://prowlarr:9696`
3. **Chaptarr Server**: `http://chaptarr:8789`
4. **API Key**: from Chaptarr's **Settings** → **General**
5. **Test**, then **Save**

## 2.3 Download clients

**Settings** → **Download Clients** — the standard arr-family clients all work. Set a dedicated category (`books` or `audiobooks`) so Chaptarr only claims its own downloads.

## 2.4 Quality profiles

Build separate profiles per media type. For audiobooks, prefer **M4B** — one chaptered file per book, which Audiobookshelf handles cleanly. Multi-file MP3 works but chapter handling is worse downstream.

Chaptarr can optionally convert MP3 audiobooks into a single chaptered M4B using [m4b-tool](https://github.com/sandreas/m4b-tool), preserving existing chapters or inserting them where none exist. Enable it per profile.

## 2.5 Renaming for Audiobookshelf

Set your naming tokens so the output matches what Audiobookshelf expects to parse:

```
{Author Name}/{Series Title}/Vol {Series Number} - {Book Title} ({Release Year})
```

