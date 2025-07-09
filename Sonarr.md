---
title: Sonarr
description: A guide to installing Sonarr in TrueNAS Scale as well as docker via compose
published: true
date: 2025-07-09T10:06:49.419Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:32:51.765Z
---

![Sonarr logo](/sonarr.png){style="height:1.5rem"}

> ## Quick-Start Checklist {.callout}
>
> 1. **Deploy container** (Docker Compose *or* TrueNAS chart).
> 2. **Create `/media/tv` root folder** in Sonarr.
> 3. **Add qBittorrent** as Download Client.
> 4. *(Optional)* Import Recyclarr profiles & advanced cleanup.

---

# What is Sonarr?

Sonarr is a PVR for Usenet and BitTorrent users.<br>
It monitors RSS feeds for new episodes, grabs, sorts, and renames them, then upgrades quality when better releases appear.

---

# 1 · Deploy Sonarr

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

### Permissions & Folder Structure {.is-success}

* **PUID / PGID** – use the media-owner UID/GID (TrueNAS SCALE default **568:568**).
* **Volumes** – keep configs in `/mnt/tank/configs/sonarr` and mount your media at `/mnt/tank/media`.<br>
  📌 See the [Folder-Structure](/Folder-Structure) guide.

---

## TrueNAS

![TrueNAS install](/screen_shot_2023-12-08_at_3.04.39_pm.png)

1. Launch the **Sonarr** app from *Community Apps* (choose **Community** image).
2. Set **Config Storage Type → Host Path**.
3. Under **Additional Storage** mount your media dataset.

---

# 2 · First-Run Configuration

## 2.1 Root Folder

1. **Settings → Media Management → Add Root Folder**
2. Pick `/media/tv` (matches the Compose/TrueNAS volume).

## 2.2 Download Client

1. **Settings → Download Client → ➕ → qBittorrent**
2. Fill the form:

| Field                   | Example        |
| ----------------------- | -------------- |
| Host                    | `10.251.0.244` |
| Port                    | `10095`        |
| Username                | `admin`        |
| Password                | •••••••••      |
| Category                | `tv-sonarr`    |
| Recent / Older Priority | **Last**       |
| Remove Completed        | ✅              |

> **Tip:** Use a dedicated qBittorrent category (e.g. `tv-sonarr`) to avoid clashing with non-Sonarr downloads.

---

# 3 · Advanced Tweaks *(optional)*

> **Warning** – Only if you sync settings with Recyclarr. Enable **Show Advanced** first. {.is-warning}

### Media-Management Presets

| Field                           | Recommended Value                      |
| ------------------------------- | -------------------------------------- |
| Rename Episodes                 | **True**                               |
| Standard / Daily / Anime Format | *TRaSH template strings*               |
| Series Folder Format            | `{Series TitleYear} [imdbid-{ImdbId}]` |
| Propers & Repacks               | **Do Not Prefer**                      |
| Set Permissions                 | **True** (chmod **777** folders)       |

### Profiles & Quality

Delete built-ins, keep only Recyclarr profiles, then set Jellyseerr default to the new ones.

### Metadata & Backups

Enable **Kodi/Emby** metadata.<br>
Set backups to `/media`, **Interval = 1 day**, **Retention = 7**.

---

# 4 · Troubleshooting

<details>
<summary><strong>Sonarr cannot see media files</strong></summary>

```bash
ls -lah /mnt/tank/media/tv
chown -R 568:568 /mnt/tank/media/tv
```

</details>

<details>
<summary><strong>Permission denied</strong></summary>

```bash
chmod -R 770 /mnt/tank/media/tv
```

</details>

<details>
<summary><strong>Downloads stay in qBittorrent</strong></summary>

* Verify **Download Client Path Mapping** matches container paths.
* Confirm Sonarr can access the completed-downloads directory.

</details>

---

# Video Guide

[![Promo card](/2025-03-24-advanced-media-management-with-s-promo-card.png)](https://www.patreon.com/posts/advanced-media-124639393)

[⇧ Back to top](#what-is-sonarr){.back-top}
