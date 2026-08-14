---
title: Audiobookshelf
description: A guide to deploying Audiobookshelf
published: true
date: 2026-08-14T11:19:57.749Z
tags: 
editor: markdown
dateCreated: 2026-08-14T11:19:57.749Z
---

# <img src="/audiobookshelf.png" class="tab-icon"> What is Audiobookshelf?

**Audiobookshelf** is a self-hosted audiobook and podcast server. It streams every common audio format on the fly, tracks listening progress per user across devices, auto-downloads podcast episodes from RSS, and serves ebooks alongside your audiobooks. Multi-user support with per-library permissions means the whole household can share one server without sharing a position in a book.

It is fully open source, including the Android and iOS apps, so nothing about your library is locked behind a vendor.

> Audiobookshelf is a **library server**, not a downloader. It reads whatever is on disk and serves it. Pair it with [Chaptarr](/chaptarr) if you want automated acquisition and renaming.
{.is-info}

# 1 · Deploy Audiobookshelf
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

```yaml
services:
  audiobookshelf:
    image: ghcr.io/advplyr/audiobookshelf:2.36.0
    container_name: audiobookshelf
    restart: unless-stopped
    user: "568:568"
    ports:
      - "13378:80"
    environment:
      - AUDIOBOOKSHELF_UID=568
      - AUDIOBOOKSHELF_GID=568
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/audiobookshelf/config:/config
      - /mnt/tank/configs/audiobookshelf/metadata:/metadata
      - /mnt/tank/media:/media
```

Create the directories with the right ownership **before** the first start:

```bash
mkdir -p /mnt/tank/configs/audiobookshelf/{config,metadata}
mkdir -p /mnt/tank/media/books/{audiobooks,ebooks}
chown -R 568:568 /mnt/tank/configs/audiobookshelf /mnt/tank/media/books
```

Browse to `http://YOUR-SERVER-IP:13378` and create the root account on first launch.



## <img src="/truenas.png" class="tab-icon"> TrueNAS

Audiobookshelf is in the **Community** train of the TrueNAS Apps catalog. Requires TrueNAS 24.10.2.2 or newer.

1. Navigate to **Apps** → **Discover Apps** in the TrueNAS UI
2. Search for **Audiobookshelf** and click **Install**
3. Configure the following:
   - **User and Group ID**: `568` / `568`
   - **Web Port**: `13378`
   - **Config Storage**: Host Path → `/mnt/tank/configs/audiobookshelf/config`
   - **Metadata Storage**: Host Path → `/mnt/tank/configs/audiobookshelf/metadata`
   - **Additional Storage**: Host Path → `/mnt/tank/media` mounted at `/media`
4. Click **Install** and wait for the app to report **Running**



# 2 · Configuration

## 2.1 Create your libraries

1. Click the account icon → **Settings** → **Libraries** → **Add Library**
2. Name it, pick the **Book** media type, and set the folder to `/media/books/audiobooks`
3. Repeat for ebooks at `/media/books/ebooks`, or add it as a second folder on the same library

Podcasts need their own library with the **Podcast** media type — books and podcasts cannot share one.

## 2.2 Folder structure

Audiobookshelf reads author, series, title, volume number, and publish year from your folder names:

```
/media/books/audiobooks/
└── Brandon Sanderson/
    └── Mistborn/
        └── Vol 1 - The Final Empire (2006)/
            └── the-final-empire.m4b
```



If your tagging is solid, you can skip all of this — go to **Settings** → **Libraries** → edit → **Scanner** and enable *Prefer audio metadata* and *Prefer OPF file* so ID3 tags win over folder names.

## 2.3 Format guidance

| Format | Notes |
|--------|-------|
| M4B | Preferred. One file per book with embedded chapters |
| MP3 (single) | Works. Chapters only if embedded |
| MP3 (multi-file) | Works. Each file becomes a chapter, so filenames must sort correctly |
| M4A, FLAC, OPUS, OGG | Supported |


## 2.4 Automatic scanning

**Settings** → **Libraries** → edit → enable **Automatically watch for changes**. New folders get picked up without a manual scan. The watcher is reliable on a local ZFS dataset; over an SMB or NFS mount it is not, so fall back to scheduled scans there.


