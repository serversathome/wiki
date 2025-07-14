---
title: Sonarr
description: A guide to installing Sonarr in TrueNAS Scale as well as docker via compose
published: true
date: 2025-07-14T13:14:17.331Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:32:51.765Z
---

# ![Sonarr](/sonarr.png){class="tab-icon"} What is Sonarr?

**Sonarr** is a TV-series PVR for Usenet and BitTorrent users. It monitors RSS feeds for new episodes, grabs, sorts, and renames them, and upgrades quality when better releases appear.

# 1 · Deploy Sonarr

> **Start by setting up your datasets, then choose your install method.**

# tabs {.tabset}

## 📂 Folder Setup

> Create the following datasets in **TrueNAS** or match these paths in **Docker volumes**. {.is-info}

| Dataset               | Mount Path in App | Description             |
| --------------------- | ----------------- | ----------------------- |
| `tank/configs/sonarr` | `/config`         | Stores Sonarr's config  |
| `tank/media`          | `/media`          | Shared media mount      |
| `tank/media/tv`       | `/media/tv`       | Folder for TV downloads |

```text
/mnt/tank/
├── configs/
│   └── sonarr/
└── media/
    └── tv/
```

> 🔒 Set ownership to `apps(568):apps(568)` (default user/group for SCALE apps). This ensures Sonarr can read/write configs and media. 

## <img src="/docker.png" class="tab-icon"> Docker Compose

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
      - 127.0.0.1:8989:8989
    restart: unless-stopped
```

> **Behind a reverse-proxy?** Expose port **8989** on `127.0.0.1` and route through Nginx Proxy Manager or Cloudflare Tunnel.

## <img src="/truenas.png" class="tab-icon"> TrueNAS Community Edition

> Use the official TrueNAS Sonarr app with custom host paths.

| Step | Action                                                        |
| ---- | ------------------------------------------------------------- |
| 1    | Apps → Discover Apps → Sonarr → Install                       |
| 2    | Set **Port** to 8989                                          |
| 3    | Sonarr Config → Host Path → `/mnt/tank/configs/sonarr`        |
| 4    | Additional Storage → Host Path → `/mnt/tank/media` → `/media` |
| 5    | Click **Save** → **Deploy**                                   |

## <img src="/nginx-proxy-manager.png" class="tab-icon"> NGINX Reverse Proxy

> Configure a reverse proxy (subdirectory or subdomain). Prefer a GUI? See [NGINX Proxy Manager](/nginx) or [Cloudflare Tunnel](/CloudflareTunnels).

<details><summary>Subdirectory <code>/sonarr</code></summary>

```nginx
location ^~ /sonarr {
  proxy_pass http://127.0.0.1:8989;
  proxy_set_header Host $host;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection $http_connection;
}
```

</details>

<details><summary>Subdomain <code>sonarr.yourdomain.tld</code></summary>

```nginx
server {
  listen 80;
  server_name sonarr.yourdomain.tld;

  location / {
    proxy_pass http://127.0.0.1:8989;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $http_connection;
  }
}
```

</details>

# 2 · First-Run Configuration

> ⚙️ **Quickly set up your library, download client, and indexers.**

# tabs {.tabset}

## 📁 Library Setup

<details open><summary><strong>Media Management</strong></summary>
  
- Enable **Rename Episodes** & **Use Hard Links**
- Add `.srt` to **Import Extra Files**

</details>

<details><summary><strong>Root Folders</strong></summary>
  
- Add `/media/tv` as a monitored root
- Ensure Sonarr user has read/write access

</details>

## 📥 Clients & Indexers

<details open><summary><strong>qBittorrent & Prowlarr</strong></summary>

| Client      | Host         | Port  | Category  | Remove Completed |
| ----------- | ------------ | ----- | --------- | ---------------- |
| qBittorrent | 10.251.0.244 | 10095 | tv-sonarr | ✅                |

1. Map `/downloads` → `/media` via **Path Translation**.
2. Sonarr → Settings → Apps → **+** → Prowlarr → Test → Save.

</details>

## 🎯 Quality Profiles

<details open><summary><strong>Profiles</strong></summary>
  
- Create or customize quality profiles
- Remove unused defaults
- Set a **Cutoff** for desired quality

</details>

# 3 · System & Maintenance

> 🛠️ **System health, tasks, backups & settings in one place.**

# tabs {.tabset}

## ⚙️ Status & Health

* **System**: .NET/Mono, SQLite ≥3.9, MediaInfo, clock sync, permissions
* **Clients**: reachable, CDD enabled, path mapping
* **Indexers**: RSS & Search enabled, avoid Jackett `/all`
* **Roots & Lists**: paths exist & accessible

Click warnings for remediation or logs.

## 📊 Tasks & Queue

* **Scheduled**: updates, backups, health checks, etc.
* **Queue**: active downloads & history
* Icons: Retry, Remove, Blocklist, Manual Import

## 💾 Backup & Updates

* **Backup**: Manual snapshot, restore, delete (System → Backups)
* **Updates**: view/install versions (System → Updates)

## ⚙️ Settings Overview

| Section             | Purpose                        |
| ------------------- | ------------------------------ |
| Media Management    | Roots, imports, naming         |
| Profiles            | Quality, delay, custom formats |
| Indexers & Clients  | RSS/Search, download setup     |
| Remote Path Mapping | Map remote → local paths       |
| Connect             | Notifications & integrations   |
| Metadata & UI       | NFO, calendar, appearance      |
| Analytics & Updates | Usage stats, update channels   |
| Backups             | Automated settings backups     |

Toggle **Show Advanced** and click **Save** when done.


# 4 · Advanced Tweaks *(Optional)*

> 🧪 For users running Recyclarr or tuning quality control.

### 🛠️ Media Management Presets

| Field                | Recommended                                                                                                 |
| -------------------- | ----------------------------------------------------------------------------------------------------------- |
| Rename Episodes      | True                                                                                                        |
| Episode Formats      | [TRaSH template strings](https://trash-guides.info/Sonarr/Sonarr-recommended-naming-scheme/#episode-format) |
| Series Folder Format | {Series TitleYear} \[imdbid-{ImdbId}]                                                                       |
| Propers & Repacks    | Do Not Prefer                                                                                               |
| Set Permissions      | True *(chmod 770)*                                                                                          |

<details><summary><strong>🏷️ Common Tags / Custom Formats</strong></summary>

| Tag         | Purpose                   |
| ----------- | ------------------------- |
| x265 / HEVC | Prefer modern video codec |
| HDR10 / DV  | Force HDR releases        |
| Atmos       | Require Dolby Atmos audio |
| Anime       | Anime-specific profiles   |

</details>

### 📐 Profiles & Quality

* Delete default profiles
* Keep Recyclarr-generated profiles
* Set Jellyseerr as default where needed

### 💾 Metadata & Backups

* Enable **Kodi/Emby** metadata
* Backup folder: `/media`, Interval: **1 day**, Retention: **7**

<details><summary><strong>📤 Restoring a Backup</strong></summary>

| Step | Action                                                                 |
| ---- | ---------------------------------------------------------------------- |
| 1    | Stop Sonarr container                                                  |
| 2    | Copy latest `.zip` from `/media/Backups` to `/mnt/tank/configs/sonarr` |
| 3    | Sonarr: **System → Backup → Restore** → Choose file                    |
| 4    | Restart Sonarr and verify settings/series                              |

</details>

<details><summary><strong>📚 Running Multiple Instances</strong></summary>

> Manage **1080p & 4K** libraries separately.

**Requirements:**

* Separate `/config` per instance
* Unique external ports (e.g. 8989, 7879)
* Distinct root folders, categories, and names

**Docker Example:**

```yaml
services:
  sonarr-4k:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr-4k
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/sonarr4k:/config
      - /mnt/tank/media-4k:/media
    ports:
      - 7879:8989
    restart: unless-stopped
```

> 🔄 You can sync instances via **Lists → Import → Sonarr**. {.is-info}

</details>

---

# 5 · Troubleshooting

> 🧯 **Start with the Health tab** — Sonarr flags missing paths, failed downloads, and indexer issues. {.is-info}

<details><summary><strong>📂 Sonarr cannot see media files</strong></summary>

```bash
ls -lah /mnt/tank/media/tv
chown -R 568:568 /mnt/tank/media/tv
```

</details>

<details><summary><strong>🔐 Permission denied</strong></summary>

```bash
chmod -R 770 /mnt/tank/media/tv
```

</details>

<details><summary><strong>📦 Downloads stay in qBittorrent</strong></summary>

* Verify **Download Client Path Mapping** matches container paths.
* Confirm Sonarr can access completed-downloads directory.

</details>

---

## ✏️ Editors & Contributors

* **Scar13t** — Page Layout & Design

> 🤝 Want to help? Open a PR or ping us on Discord!

---

# 6 · Video Guide

[![Watch on Patreon](/2025-03-24-advanced-media-management-with-s-promo-card.png)](https://www.patreon.com/posts/advanced-media-124639393)

[⇗ Back to top](#what-is-sonarr){.back-top}
