---
title: Sonarr
description: A guide to installing Sonarr in TrueNAS Scale as well as docker via compose
published: true
date: 2025-07-28T13:27:21.318Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:32:51.765Z
---

# ![Sonarr](/sonarr.png){.tab-icon} What is Sonarr?

**Sonarr** is a TV-series PVR for Usenet and BitTorrent users. It monitors RSS feeds for new episodes, grabs, sorts, and renames them, and upgrades quality when better releases appear.

# 1 · Deploy Sonarr

Start by setting up your datasets, then choose your install method.

# tabs {.tabset}

## 📂 Folder Setup

> Create the following datasets in **TrueNAS** or match these paths in **Docker volumes**. {.is-info}

<div class="table-scroll">

| Dataset               | Mount Path in App | Description             |
| --------------------- | ----------------- | ----------------------- |
| `tank/configs/sonarr` | `/config`         | Stores Sonarr's config  |
| `tank/media`          | `/media`          | Shared media mount      |
| `tank/media/tv`       | `/media/tv`       | Folder for TV downloads |

</div>

```text
/mnt/tank/
├── configs/
│   └── sonarr/
└── media/
    └── tv/
```

> 🔒 Set ownership to `apps(568):apps(568)` (default user/group for SCALE apps). This ensures Sonarr can read/write configs and media. {.is-success}

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
      - 8989:8989
    restart: unless-stopped
```

## <img src="/truenas.png" class="tab-icon"> TrueNAS Community Edition

Use the official TrueNAS Sonarr app with custom host paths.

| Step | Action                                                        |
| ---- | ------------------------------------------------------------- |
| 1    | Apps → Discover Apps → **Sonarr** → **Install**               |
| 2    | Set **Port** to **8989**                                      |
| 3    | Sonarr Config → Host Path → `/mnt/tank/configs/sonarr`        |
| 4    | Additional Storage → Host Path → `/mnt/tank/media` → `/media` |
| 5    | Click **Save** → **Deploy**                                   |


## <img src="/nginx-proxy-manager.png" class="tab-icon"> NGINX Reverse Proxy

> Configure a reverse proxy (subdirectory or subdomain). Prefer a GUI? See [NGINX Proxy Manager](/nginx) or [Cloudflare Tunnel](/CloudflareTunnels). {.is-info}

<details><summary>Subdirectory <code>/sonarr</code></summary>

<details class="code-block"><summary>Nginx location block</summary>

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

</details>

<details><summary>Subdomain <code>sonarr.yourdomain.tld</code></summary>

<details class="code-block"><summary>Nginx server block</summary>

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

</details>


# 2 · First-Run Configuration

> ⚙️ **Quickly set up your library, download client, and indexers.** {.is-success}

# tabs {.tabset}

## 📁 Library Setup

* Enable **Rename Episodes** & **Use Hard Links**

**Root Folders**
* Add `/media/tv` as a monitored root
* Ensure Sonarr user has read/write access



## 📥 Clients & Indexers

**qBittorrent & Prowlarr**


| Client      | Host         |  Port | Category  | Remove Completed |
| ----------- | ------------ | ---- | --------- | --------------- |
| qBittorrent | 192.168.1.25 | 8080 | tv-sonarr | ✅ |


## 🎯 Quality Profiles

**Profiles**

* Create or customize quality profiles
* Remove unused defaults
* Set a **Cutoff** for desired quality


# 3 · Advanced Tweaks *(Optional)*

🧪 For users running Recyclarr or tuning quality control.

### 🛠️ Media Management Presets

<div class="table-scroll">

| Field                | Recommended                                                                                                 |
| -------------------- | ----------------------------------------------------------------------------------------------------------- |
| Rename Episodes      | True                                                                                                        |
| Episode Formats      | [TRaSH template strings](https://trash-guides.info/Sonarr/Sonarr-recommended-naming-scheme/#episode-format) |
| Series Folder Format | {Series TitleYear} \[imdbid-{ImdbId}]                                                                       |
| Propers & Repacks    | Do Not Prefer                                                                                               |
| Set Permissions      | True *(chmod 770)*                                                                                          |

</div>


### 📐 Profiles & Quality

* Delete default profiles
* Keep Recyclarr-generated profiles
* Set Jellyseerr as default where needed



### 💾 Metadata & Backups

* Enable **Kodi/Emby** metadata
* Backup folder: `/media`, Interval: **1 day**, Retention: **7**

<details><summary><strong>📤 Restoring a Backup</strong></summary>
  
1. Navigate to **System → Backup**
1. Use one of two options for restoration:
a. Either restore from a backup in the configured folder by clicking the clock icon at the end of a row
b.  Click the **Restore Backup** icon in the top to restore from a local .zip backup

</details>

<details><summary><strong>📚 Running Multiple Instances</strong></summary>

**Manage 1080p & 4K libraries separately**

**Requirements:**

* Separate `/config` per instance
* Unique external ports (e.g. 8988, 7879)
* Distinct root folders, categories, and names

<details class="code-block"><summary>Docker example</summary>

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
      - 8988:8989
    restart: unless-stopped
```

</details>

> 🔄 You can sync instances via **Lists → Import → Sonarr**. {.is-info}

</details>

---

# 4 · Troubleshooting

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

## ✏️ Editors & Contributors

* **Scar13t** — Page Layout & Design

> 🤝 Want to help? Open a PR or ping us on Discord!
{.is-success}


---

<div class="video-grid">

  <div class="video-card">
    <div class="video-platform">
      <img src="/patreon-light.png" alt="Patreon Logo">
      <span>Patreon Video</span>
    </div>
    <a href="https://www.patreon.com/posts/advanced-media-124639393" target="_blank">
      <img src="/2025-03-24-advanced-media-management-with-s-promo-card.png" class="video-thumbnail" alt="Advanced Media Management on Patreon">
    </a>
    <div class="video-title">Advanced Media Management</div>
  </div>

<div class="video-card">
  <div class="video-platform">
    <img src="/youtube.png" alt="YouTube Logo">
    <span>YouTube Guide</span>
  </div>
  <a href="https://youtu.be/gp06bncv8zs" target="_blank" rel="noopener noreferrer">
    <img src="https://img.youtube.com/vi/gp06bncv8zs/hqdefault.jpg" class="video-thumbnail" alt="Installing Sonarr TrueNAS Community Edition 2025">
  </a>
  <div class="video-title">Installing Sonarr (TrueNAS CE 2025)</div>
</div>


</div>


