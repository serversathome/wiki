---
title: ROMarr
description: A guide to deploying ROMarr
published: true
date: 2026-10-31T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-10-31T12:00:00.000Z
---

# <img src="/romarr.png" class="tab-icon"> What is ROMarr?

**ROMarr** is "the \*arr for games". It works like Sonarr or Radarr, but for retro and console ROMs: it searches your indexers, sends releases to your download client, verifies the files against No-Intro and Redump checksums and imports them into your library. It pairs perfectly with [RomM](/romm), which it rescans automatically after every import.

- Search through Prowlarr, Torznab/Newznab indexers or torrent RSS
- Works with qBittorrent, Transmission, Deluge, SABnzbd, NZBGet, debrid services and more
- Imports into RomM, Gaseous, Retrom or a plain folder for ES-DE, Batocera, EmuDeck or LaunchBox
- Checksum verification against No-Intro and Redump DATs, with collection gap reports and 1G1R region selection
- Duplicate audit, preview-first renaming with rollback and IGDB/RAWG metadata for the calendar and discover pages

<div class="glance">
  <div><span>Port</span><b><code>6868</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/BlizzHacker/romarr"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy ROMarr
```yaml
services:
  romarr:
    image: ghcr.io/blizzhacker/romarr:1.8.0
    container_name: romarr
    restart: unless-stopped
    ports:
      - 6868:6868
    environment:
      - PUID=568
      - PGID=568
      - UMASK=002
      - TZ=America/New_York
      - LIBRARY_KIND=romm
      - LIBRARY_URL=http://YOUR-SERVER-IP:8080 # your RomM URL
      - LIBRARY_PATH=/roms
    volumes:
      - /mnt/tank/configs/romarr:/config
      - /mnt/tank/media/romm/library/roms:/roms
      - /mnt/tank/media/downloads:/downloads
```

1. Point the `/roms` volume at the folder your library scans. The path above matches the [RomM guide](/romm), where platforms live in `library/roms/{platform}`
1. Set `LIBRARY_URL` to your RomM address. Not using RomM? Set `LIBRARY_KIND=folder` and remove `LIBRARY_URL`
1. Deploy the stack and open `http://YOUR-SERVER-IP:6868`

> The `/downloads` path must match what your download client reports, character for character. If qBittorrent saves to `/downloads/complete/game.zip`, ROMarr must see the same file at that path. Otherwise add a remote path mapping under **Settings → Media Management**.
{.is-warning}

> The library and connection variables only seed the first start. After that, the **Settings** pages are in charge and changes to the compose file are ignored.
{.is-info}

# 2 · Configuration

## 2.1 Claim Your Install
1. The first visit asks you to set a password. There are no default credentials
1. Do this right after deploying, since a fresh install belongs to whoever opens it first
1. Optional: enable TOTP two-factor login under **Settings → General**

## 2.2 Connect Your Library
1. Go to **Settings → Libraries**
1. For RomM, enter the URL plus a username and password, then click **Test**

> Create a dedicated RomM user for ROMarr instead of using your admin account. It only needs to read the library and trigger a rescan, so give it the task permission in RomM or imports will never show up.
{.is-info}

## 2.3 Download Clients and Indexers
1. **Settings → Download Clients**: add qBittorrent, SABnzbd or another client and click **Test**. Each protocol needs its own client, so adding qBittorrent does not cover Usenet releases
1. **Settings → Indexers**: add your Prowlarr URL and API key, or add Torznab/Newznab indexers directly
1. Use **Interactive Search** on a game for your first grab and follow it in **Queue** and **History**

> Already have ROMs on disk? The **Setup Guide** under Settings has an "I already have ROMs" path that scans `/downloads` and imports what it finds, no indexer needed.
{.is-success}

# <img src="/youtube.png" class="tab-icon"> 3 · Video

*No video yet, check back soon!*
