---
title: Chaptarr
description: A guide to deploying Chaptarr
published: true
date: 2026-08-14T11:11:39.002Z
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

> **Do not set `user:` in this compose file.** Unlike most containers, Chaptarr's entrypoint needs to start as root to fix permissions before dropping to `PUID`/`PGID`. Setting `user:` bypasses that entirely. Use the environment variables only.
{.is-warning}

> Without `PUID`/`PGID` the image defaults to `99:100` (the Unraid convention), not `568`. Always set them explicitly on TrueNAS.
{.is-warning}

Verify the app user can actually write to the library before you configure anything:

```bash
docker exec -u 568:568 chaptarr sh -c 'touch /media/books/audiobooks/.t && rm /media/books/audiobooks/.t && echo ok'
```

> There is no TrueNAS catalog app for Chaptarr. Deploy it as a Dockge stack or a TrueNAS **Custom App**.
{.is-info}

# 2 · Configuration

## 2.1 Root folders

**Settings** → **Media Management** → **Root Folders**:

| Library | Path |
|---------|------|
| Audiobooks | `/media/books/audiobooks` |
| Ebooks | `/media/books/ebooks` |


You can keep them separate or place ebooks alongside their audiobook counterparts — Chaptarr supports both layouts.

## 2.2 The single-mount rule

Chaptarr and your download client must see the **same path string** for completed downloads. Mounting `/mnt/tank/media` to `/media` in both containers is what makes imports instant hardlinks instead of full-file copies.

> Get this wrong and imports still work — they just silently duplicate every book on disk and burn the seeding copy. This is the single most common mistake when adding an arr to an existing stack.
{.is-danger}

```mermaid
graph LR
    A[Prowlarr] -->|indexer results| B[Chaptarr]
    B -->|send release| C[qBittorrent / SABnzbd]
    C -->|/media/downloads| B
    B -->|hardlink + rename| D[/media/books/audiobooks]
    D -->|watcher picks up| E[Audiobookshelf]
```

## 2.3 Indexers

Chaptarr is not a named application in Prowlarr. Add it there as a **Readarr**-type app — the API is compatible and syncing works out of the box.

1. In Prowlarr: **Settings** → **Apps** → **+** → **Readarr**
2. **Prowlarr Server**: `http://prowlarr:9696`
3. **Chaptarr Server**: `http://chaptarr:8789`
4. **API Key**: from Chaptarr's **Settings** → **General**
5. **Test**, then **Save**

## 2.4 Download clients

**Settings** → **Download Clients** — the standard arr-family clients all work. Set a dedicated category (`books` or `audiobooks`) so Chaptarr only claims its own downloads.

## 2.5 Quality profiles

Build separate profiles per media type. For audiobooks, prefer **M4B** — one chaptered file per book, which Audiobookshelf handles cleanly. Multi-file MP3 works but chapter handling is worse downstream.

Chaptarr can optionally convert MP3 audiobooks into a single chaptered M4B using [m4b-tool](https://github.com/sandreas/m4b-tool), preserving existing chapters or inserting them where none exist. Enable it per profile.

## 2.6 Renaming for Audiobookshelf

Set your naming tokens so the output matches what Audiobookshelf expects to parse:

```
{Author Name}/{Series Title}/Vol {Series Number} - {Book Title} ({Release Year})
```

## 2.7 Optional PostgreSQL

Chaptarr defaults to SQLite. To use Postgres instead, set at minimum `Chaptarr__Postgres__Host` plus credentials.

| Variable | Default |
|----------|---------|
| `Chaptarr__Postgres__Host` | — |
| `Chaptarr__Postgres__Port` | `5432` |
| `Chaptarr__Postgres__User` | — |
| `Chaptarr__Postgres__Password` | — |
| `Chaptarr__Postgres__MainDb` | `chaptarr-main` |
| `Chaptarr__Postgres__LogDb` | `chaptarr-log` |
| `Chaptarr__Postgres__CacheDb` | `chaptarr-cache` |
{.dense}

> Chaptarr does **not** create the databases for you. Create all three and grant the configured user access before starting the container.
{.is-warning}

# 3 · Security and privacy

Keep Chaptarr on the LAN. It stores indexer and download client credentials, and the hardening below is newly added rather than battle-tested.

The fork adds several things Readarr never had: constant-time API key comparison, login brute-force throttling, HSTS/CSP/X-Frame-Options headers, image proxy target validation, secret redaction in API responses, no analytics or crash reporting, and SHA256 verification of update binaries.

> Full backups contain the database and config **including credentials** in an unencrypted zip. Never upload one to cloud storage without encrypting it separately. Chaptarr offers optional passphrase-encrypted Quickstart Settings Backups as an alternative.
{.is-danger}

> Metadata and matching requests go to `api2.chaptarr.com` and may include provider IDs, search text, media type, tags, and the filename being matched. Full paths, user identity, and indexer or download client credentials are not sent. Update checks send version, OS, architecture, and runtime info.
{.is-info}

# 4 · Reference

| Setting | Value |
|---------|-------|
| Image | `chaptarr/chaptarr:latest` |
| Web UI | `http://SERVER-IP:8789` |
| Internal port | `8789` |
| Config path | `/mnt/tank/configs/chaptarr` |
| Media path | `/mnt/tank/media` → `/media` |
| PUID / PGID | `568` / `568` |
| License | GPL-3.0 |
{.dense}

> There are no semver image tags yet, so `:latest` is the only practical option. Pin by digest after your first pull if you want reproducible deploys.
{.is-info}

- [GitHub *Chaptarr/chaptarr*](https://github.com/Chaptarr/chaptarr)
- [Chaptarr Wiki *wiki.chaptarr.com*](https://wiki.chaptarr.com)
- [Discord *community support*](https://discord.gg/G9ZbgWS5rp)
{.links-list}

> Chaptarr is an independent community project. It is **not** affiliated with or supported by the Servarr team — do not send Chaptarr issues to Sonarr, Radarr, or Prowlarr. The maintainers also disclose that development is assisted by AI tooling.
{.is-info}

# <img src="/youtube.png" class="tab-icon"> 5 · Video

https://youtu.be/VIDEO-ID-HERE