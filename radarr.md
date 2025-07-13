---
title: Radarr
description: A guide to installing Radarr in TrueNAS Scale as well as docker via compose
published: true
date: 2025-07-13T21:06:45.400Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:32:11.647Z
---

# ![Radarr](/radarr.png){class="tab-icon"} What is Radarr?

Radarr is a **movie collection manager** for Usenet and BitTorrent. It monitors RSS feeds for new releases, interfaces with clients/indexers to grab, sort, and rename movies, and can upgrade quality automatically when better releases appear.

---

<details class="quickstart" open>
<summary><strong>🚀 Quick‑Start Checklist</strong></summary>

1. **Deploy container** (Docker Compose *or* TrueNAS Apps)
2. **Create** `/media/movies` **root folder** in Radarr (Monitored → ✅)
3. **Add qBittorrent** as Download Client
4. **Add Indexers via Prowlarr** so Radarr can actually find releases
5. *(Optional)* Import Recyclarr profiles & advanced cleanup

</details>

---

# 1 · Deploy Radarr

# tabs {.tabset}

## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/radarr:/config
      - /mnt/tank/media:/media
    ports:
      - 7878:7878
    restart: unless-stopped
```

### Permissions & Folder Structure {.is-success}

* **PUID / PGID** – media‑owner UID/GID (TrueNAS SCALE default **568:568**).
* **Volumes** – configs at `/mnt/tank/configs/radarr`, media at `/mnt/tank/media`.
  📌 See the [Folder‑Structure](/Folder-Structure) guide.

> **Behind a reverse‑proxy?** Expose port **7878** only on `127.0.0.1` and route externally via Nginx Proxy Manager or Cloudflare Tunnel.

---

## <img src="/truenas.png" class="tab-icon"> TrueNAS Community Edition

|  Step  |  Action                                                                         |
| ------ | ------------------------------------------------------------------------------- |
| **1**  | **Apps → Discover Apps → Radarr → Install**                                     |
| **2**  | **Port Number → 7878**                                                          |
| **3**  | **Radarr Config Storage → Host Path** → `/mnt/tank/configs/radarr`              |
| **4**  | **Additional Storage → Host Path** → mount dataset `/mnt/tank/media` ➜ `/media` |
| **5**  | Click **Save → Deploy**                                                         |

---

# 2 · First‑Run Configuration

## 2.1 Root Folder  <span class="chip">Mandatory</span>

1. **Settings → Media Management → Add Root Folder**
2. Choose **/media/movies** and ensure it’s **Monitored** (green ✔️).

## 2.2 Download Client  <span class="chip">qBittorrent</span>

1. **Settings → Download Client → ➕ → qBittorrent**
2. Fill the form:

|  Field                  |  Example         |
| ----------------------- | ---------------- |
|  Host                   |  `10.251.0.244`  |
|  Port                   |  `10095`         |
|  Username               |  `admin`         |
|  Password               |  ••••••••        |
|  Category               |  `movies-radarr` |
|  Recent/Older Priority  |  **Last**        |
|  Remove Completed       |  ✅               |

> **Remote downloader?** Use **Path Translation** to map `/downloads` (qBittorrent) → `/media` (Radarr).

## 2.3 Indexers (via Prowlarr)

1. Connect Radarr in **Prowlarr → Settings → Apps → +**.
2. Add indexers (Jackett, Torznab, etc.).
3. **Test → Save** — Radarr inherits them automatically.

---

# 3 · Advanced Tweaks *(optional)*

> **Warning** – For Recyclarr users. Enable **Show Advanced** first. {.is-warning}

### Media‑Management Presets

|  Field                  |  Recommended           |
| ----------------------- | ---------------------- |
|  Rename Movies          |  `True`                |
|  Standard Movie Format  |  *TRaSH Radarr string* |
|  Propers & Repacks      |  `Do Not Prefer`       |
|  Set Permissions        |  `True` *(chmod 777)*  |

<details><summary><strong>📑 Common Tags / Custom Formats (cheat‑sheet)</strong></summary>

|  Tag          |  Purpose                    |
| ------------- | --------------------------- |
|  x265 / HEVC  |  Prefer modern video codec  |
|  HDR10 / DV   |  Force HDR releases         |
|  Atmos        |  Require Dolby Atmos audio  |
|  Anime        |  Anime‑specific profiles    |

</details>

### Profiles & Quality

Delete default profiles → keep Recyclarr-generated profiles → set Jellyseerr default.

### Metadata & Backups

Enable **Kodi/Emby** metadata.
Backups: `/media`, **Interval = 1 day**, **Retention = 7**.

<details><summary><strong>🔄 Restoring&nbsp;a&nbsp;Backup</strong></summary>

| Step  | Action                                                                      |
| ----- | --------------------------------------------------------------------------- |
| **1** | Stop the Radarr container / chart                                           |
| **2** | Copy the latest `*.zip` from `/media/Backups` to `/mnt/tank/configs/radarr` |
| **3** | **System → Backup → Restore** inside Radarr, select the file                |
| **4** | Restart Radarr and verify library/settings                                  |

</details>

---

# 4 · Troubleshooting

<details><summary><strong>Radarr cannot see media files</strong></summary>

```bash
ls -lah /mnt/tank/media/movies
chown -R 568:568 /mnt/tank/media/movies
```

</details>

<details><summary><strong>Permission denied</strong></summary>

```bash
chmod -R 770 /mnt/tank/media/movies
```

</details>

<details><summary><strong>Downloads stay in qBittorrent</strong></summary>

* Verify **Download Client Path Mapping**.
* Confirm Radarr can access the completed-downloads directory.

</details>

---

## ✏️ Editors & Contributors

> Special thanks to **@Hydrology** and the Servers\@Home community for maintaining this guide.

---

# Video Guide

[![Promo](/2025-03-18-advanced-media-management-with-r-promo-card.png)](https://www.patreon.com/posts/advanced-media-124637606)

[⇧ Back to top](#what-is-radarr){.back-top}
