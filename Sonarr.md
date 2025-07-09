---
title: Sonarr
description: A guide to installing Sonarr in TrueNAS Scale as well as docker via compose
published: true
date: 2025-07-09T12:07:24.496Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:32:51.765Z
---

# ![](/sonarr.png){class="tab-icon"} What is Sonarr?

Sonarr is a PVR for Usenet and BitTorrent users. It monitors RSS feeds for new episodes, grabs, sorts and renames them, then upgrades quality when a better release appears.


<details class="quickstart" open>
<summary><strong>🚀 Quick‑Start Checklist</strong></summary>

1. **Deploy container** (Docker Compose *or* TrueNAS chart)
2. **Create** `/media/tv` **root folder** in Sonarr *(make sure it’s **Monitored** → ✅)*
3. **Add qBittorrent** as Download Client
4. **Add Indexers via Prowlarr** so Sonarr can actually find releases
5. *(Optional)* Import Recyclarr profiles & advanced cleanup

</details>



# 1 · Deploy Sonarr

# {.tabset}

## <img src="/docker.png" class="tab-icon"> Docker Compose

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

### Permissions & Folder Structure

* **PUID / PGID** – media‑owner UID/GID (TrueNAS SCALE default **568:568**).
* **Volumes** – configs at `/mnt/tank/configs/sonarr`, media at `/mnt/tank/media`.
  📌 See the [Folder‑Structure](/Folder-Structure) guide.

## <img src="/truenas.png" class="tab-icon"> TrueNAS Community Edition

|  Step  |  Action    |
| ------ | ----------- |
| **1**  | **Apps → Discover Apps → Sonarr → Install**      |
| **2**  | **Port Number → 8989**  |
| **3**  | **Sonarr Config Storage → Host Path** → `/mnt/tank/configs/sonarr`   |
| **4**  | **Additional Storage → Host Path** → mount dataset `/mnt/tank/media` ➜ `/media` |
| **5**  | Click **Save → Deploy**   |


# 2 · First‑Run Configuration

## 2.1 Root Folder

1. **Settings → Media Management → Add Root Folder**
2. Choose **/media/tv** and ensure the switch is set to **Monitored** ✅ 

> *If it’s Unmonitored, Sonarr will ignore new episodes!* {.is-info}

## 2.2 Download Client
1. **Settings → Download Client → ➕ → qBittorrent**
2. Fill the form:

|  Field                  |  Example        |
| ----------------------- | --------------- |
|  Host                   |  `10.251.0.244` |
|  Port                   |  `8080`        |
|  Username               |  `admin`        |
|  Password               |  ••••••••       |
|  Category               |  `tv-sonarr`    |
|  Recent/Older Priority  |  **Last**       |
|  Remove Completed       |  ✅              |


## 2.3 Indexers (via Prowlarr)

1. Install **[Prowlarr](/Prowlarr)** and connect it to Sonarr (`Settings → Apps → +`).
2. Add indexers in Prowlarr
3. Click **Test → Save** — Sonarr now inherits all indexers automatically.


# 3 · Advanced Tweaks *(optional)*

> **Warning** – For Recyclarr users. Enable **Show Advanced** first. {.is-warning}

## 3.1 Media‑Management Presets

|  Field                 |  Recommended                            |
| ---------------------- | --------------------------------------- |
|  Rename Episodes       |  `True`                                 |
|  Episode Formats       |  [TRaSH template strings](https://trash-guides.info/Sonarr/Sonarr-recommended-naming-scheme/#episode-format)    |
|  Series Folder Format  |  `{Series TitleYear} [imdbid-{ImdbId}]` |
|  Propers & Repacks     |  `Do Not Prefer`                        |
|  Set Permissions       |  `True` *(chmod 777)*                   |

<details><summary><strong>📑 Common Tags / Custom Formats (cheat‑sheet)</strong></summary>

|  Tag          |  Purpose                    |
| ------------- | --------------------------- |
|  x265 / HEVC  |  Prefer modern video codec  |
|  HDR10 / DV   |  Force HDR releases         |
|  Atmos        |  Require Dolby Atmos audio  |
|  Anime        |  Anime‑specific profiles    |

Copy these into **Settings → Profiles → Custom Formats**.

</details>

## 3.2 Profiles & Quality

Delete default profiles → keep Recyclarr‑generated profiles → set Jellyseerr default.

## 3.3 Metadata & Backups

Enable **Kodi/Emby** metadata.
Backups: `/media`, **Interval = 1 day**, **Retention = 7**.

<details><summary><strong>🔄 Restoring&nbsp;a&nbsp;Backup</strong></summary>

| Step  | Action          |
| ----- | --------------- |
| **1** | Stop the Sonarr container / chart     |
| **2** | Copy the latest `*.zip` from `/media/Backups` to your config folder (`/mnt/tank/configs/sonarr`) |
| **3** | In Sonarr: **System → Backup → Restore** → choose the file you just copied      |
| **4** | Restart Sonarr when prompted and verify your settings/series are back |

</details>

# 4 · Troubleshooting

> **Check Health tab first!** Sonarr will flag missing root paths, import errors and indexer failures in **System → Health**. {.is-info}

<details><summary><strong>Sonarr cannot see media files</strong></summary>

```bash
ls -lah /mnt/tank/media/tv
chgrp -R root:apps /mnt/tank/media/tv
```

</details>

<details><summary><strong>Permission denied</strong></summary>

```bash
chmod -R 770 /mnt/tank/media/tv
```

</details>

<details><summary><strong>Downloads stay in qBittorrent</strong></summary>

* Verify **Download Client Path Mapping** matches container paths.
* Confirm Sonarr can access the completed-downloads directory.

</details>


# 5 · Video Guide

[![](/2025-03-24-advanced-media-management-with-s-promo-card.png)](https://www.patreon.com/posts/advanced-media-124639393)
