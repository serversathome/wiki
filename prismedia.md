---
title: Prismedia
description: A guide to deploying Prismedia
published: true
date: 2026-09-03T11:49:15.948Z
tags: 
editor: markdown
dateCreated: 2026-09-03T11:49:15.948Z
---

# What is Prismedia?

**Prismedia** is a private, self-hosted media library that handles movies, series, music, audiobooks, eBooks, comics, images, and galleries in a single system — plus requesting and acquiring new media. Instead of running one app for playback, another for requests, and a suite of services for acquisition and metadata, Prismedia keeps discovery, requests, downloads, identification, organization, and playback attached to the same library item.



# <img src="/docker.png" class="tab-icon"> 1 · Deploy Prismedia

```yaml
services:
  prismedia:
    image: ghcr.io/pauljoda/prismedia:latest
    container_name: prismedia
    ports:
      - "8008:8008"
    volumes:
      - /mnt/tank/configs/prismedia:/data
      - /mnt/tank/media:/media
    restart: unless-stopped
```


# 2 · Configuration



## 2.1 Watched libraries and the first scan

Prismedia needs at least one **watched library root** before it has anything to show.

1. Go to **Settings → Watched Libraries** and add a root
2. Use the path **inside the container** — with the compose above that is `/media`, `/media/movies`, `/media/books/ebooks`, and so on
3. Enable the scan toggles that match the folder
4. A new root starts scanning immediately; you can also run scans from **Jobs**, **Settings → Watched Libraries**, or **Files**

| Toggle | Scans |
|--------|-------|
| **Videos** | Video files → movies, series, seasons, episodes |
| **Images** | Loose images and folders of images (galleries) |
| **Books** | `.cbz`/`.zip` comics, `.epub`/`.pdf` eBooks, `.m4b`/`.m4a`/`.mp3` audiobooks |
| **Audio** | Music folders → artists, albums, tracks |


Per-root settings also cover **Enabled**, **Recursive**, **NSFW** (marks everything under the root as restricted by default), and **Auto Identify**.

> Use separate roots when folders need different scan behavior. Pointing one root at `/media` with every toggle on works, but a root per media type gives you cleaner control — for example `/media/movies` (Videos), `/media/books/ebooks` and `/media/books/audiobooks` (Books), `/media/music` (Audio).
{.is-success}

Scans are incremental, so an unchanged root finishes almost instantly on the next pass. The worker queues its own follow-up jobs for probes, thumbnails, sprites, waveforms, subtitles, and HLS assets — watch them in **Jobs**.

## 2.2 Identify

Once media is scanned, open **Identify** to pull provider metadata. Run providers, review the field-by-field proposal, pick artwork, walk into child proposals (seasons/episodes, volumes/chapters, albums/tracks), and accept. Turn on **Auto Identify** per root to apply confident matches automatically during scans.

## 2.3 Requests

**Request** is the built-in acquisition workspace — the Radarr/Sonarr/Lidarr/Readarr job, in-app.

1. Add **Prowlarr** (or direct Torznab/Newznab indexers) in Settings
2. Add a download client — qBittorrent, Transmission, or SABnzbd
3. Search from **Request**, create the Wanted entity, and pick a release or let it grab
4. Track progress, Missing and Cutoff Unmet lists, and History on the same library pages

## 2.4 OPDS readers

Point any OPDS 1.2 reader at your library:

```text
http://<truenas-ip>:8008/opds
```

Authentication is HTTP Basic using your Prismedia username and password. A folder only appears in OPDS after it is a library root with **Books** scanning enabled and a completed scan.

> Some readers authenticate the catalog request but drop the header when fetching covers or downloads. If the catalog loads but covers fail, append a session token: `/opds?access_token=YOUR_TOKEN`. Treat that URL as a secret.
{.is-warning}

