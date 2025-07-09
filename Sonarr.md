---
title: Sonarr
description: A guide to installing Sonarr in TrueNAS Scale as well as docker via compose
published: true
date: 2025-07-09T09:55:52.222Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:32:51.765Z
---

![Sonarr logo](/sonarr.png)

# What is Sonarr?

Sonarr is a PVR for Usenet and BitTorrent users.<br>
It monitors RSS feeds for new episodes, grabs, sorts, and renames them, and can auto‑upgrade quality when a better version appears.

---

# Installation

# tabs {.tabset}

## Docker Compose

```yaml
services:
  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/sonarr:/config
      - /mnt/tank/media:/media
    ports:
      - 8989:8989
    restart: unless-stopped
```

### Permissions & Folder Structure

* **PUID / PGID** – ensure the user owns your media folders (TrueNAS SCALE default `568:568`).
* **Volumes** – configs under `/mnt/tank/configs/sonarr`, media under `/mnt/tank/media`.<br>
  📌 See [Folder‑Structure](/Folder-Structure) for more detail.

---

## TrueNAS

![TrueNAS install](/screen_shot_2023-12-08_at_3.04.39_pm.png)

1. Install **Sonarr** from *TrueNAS Community Apps*.
2. Choose the **Community** image when available.
3. Set **Config Storage Type → Host Path** (per [Folder‑Structure](/Folder-Structure)).
4. Under **Additional Storage** mount your media directory inside the container.

---

# Sonarr Configuration

## Root Folder

1. *Settings → Media Management* → **Add Root Folder**
2. Select `/media/tv`

## Download Client

1. *Settings → Download Client* → ➕ → **qBittorrent**
2. Configure:

| Setting     | Value                    |
| ----------- | ------------------------ |
| Host        | *Server IP*              |
| Port        | *qBittorrent WebUI port* |
| Credentials | *Set during install*     |

![qBittorrent config](/screenshot_from_2023-12-14_14-33-11.png)

---

## Advanced Settings

> **Warning** – These options assume you use Recyclarr. Enable **Show Advanced** (cog icon) first. {.is-warning}

### Media Management

| Field                   | Value                                                                                                                                                                                                                                  |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Rename Episodes         | `True`                                                                                                                                                                                                                                 |
| Standard Episode Format | `{Series TitleYear} - S{season:00}E{episode:00} - {Episode CleanTitle} [{Custom Formats }{Quality Full}]{[MediaInfo VideoDynamicRangeType]}{[Mediainfo AudioCodec}{ Mediainfo AudioChannels]}{[MediaInfo VideoCodec]}{-Release Group}` |
| Daily Episode Format    | *as above*                                                                                                                                                                                                                             |
| Anime Episode Format    | *as above*                                                                                                                                                                                                                             |
| Series Folder Format    | `{Series TitleYear} [imdbid-{ImdbId}]`                                                                                                                                                                                                 |
| Episode Title Required  | `Never`                                                                                                                                                                                                                                |
| Propers and Repacks     | `Do Not Prefer`                                                                                                                                                                                                                        |
| Recycling Bin Cleanup   | `1`                                                                                                                                                                                                                                    |
| Set Permissions         | `True`                                                                                                                                                                                                                                 |
| chmod Folder            | `777`                                                                                                                                                                                                                                  |

### Profiles

1. Delete default profiles; keep only Recyclarr‑generated ones.
2. Point Jellyseerr to the new profiles.

> **Why Recyclarr profiles?**  They automate quality filters, metadata tags and prioritisation for consistent results. {.is-info}

### Connect (Notifications)

See [Notifications](/Notifications#radarrsonarrprowlarr).

### Metadata

Enable **Kodi (XBMC) / Emby** metadata.<br>

> Sends episode info, artwork and tags to your media server for richer libraries. {.is-info}

### General → Backups

* Folder: `/media`
* Interval: `1`
* Retention: `7` days (default) or your preference.

---

# Troubleshooting

## Sonarr cannot see media files

```bash
ls -lah /mnt/tank/media/tv
```

Fix ownership if needed:

```bash
chown -R 568:568 /mnt/tank/media/tv
```

## Permission denied

```bash
chmod -R 770 /mnt/tank/media/tv
```

## Sonarr downloads but does not move files

* Verify *Download Client Path Mapping*.
* Ensure Sonarr can reach the completed‑downloads directory.

---

# Video

[![Promo](/2025-03-24-advanced-media-management-with-s-promo-card.png)](https://www.patreon.com/posts/advanced-media-124639393)
