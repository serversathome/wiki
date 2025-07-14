---
title: Sonarr
description: A guide to installing Sonarr in TrueNAS Scale as well as docker via compose
published: true
date: 2025-07-14T03:23:04.758Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:32:51.765Z
---

# ![Sonarr](/sonarr.png){class="tab-icon"} What is Sonarr?

**Sonarr** is a TV-series PVR for Usenet and BitTorrent users. It monitors RSS feeds for new episodes, grabs, sorts, and renames them, and upgrades quality when better releases appear.

> 📌 Works great with **qBittorrent**, **Prowlarr**, and **Jellyfin/Plex** for fully automated TV downloads.

---

# 1 · Deploy Sonarr

> **Start by setting up your datasets, then choose your install method.**

# tabs {.tabset}

## 📂 Folder Setup

> Create the following datasets in **TrueNAS** or match these paths in **Docker volumes**.

| Dataset               | Mount Path in App     | Description              |
|-----------------------|------------------------|---------------------------|
| `tank/configs/sonarr` | `/config`              | Stores Sonarr's config    |
| `tank/media`          | `/media`               | Shared media mount        |
| `tank/media/tv`       | `/media/tv`            | Folder for TV downloads   |

```text
/mnt/tank/
├── configs/
│   └── sonarr/
└── media/
    └── tv/
```

> 🔒 Set ownership to `apps(568):apps(568)` the default user/group used by TrueNAS SCALE apps and most containers.  
> This ensures Sonarr has full access to config and media folders.

---

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

> **Behind a reverse‑proxy?**  
Expose port **8989** only on `127.0.0.1` and route externally via Nginx Proxy Manager or Cloudflare Tunnel.

<details><summary><strong>🔁 NGINX / Apache Reverse Proxy</strong></summary>

### NGINX (Subdirectory: `/sonarr`)

```nginx
location ^~ /sonarr {
    proxy_pass http://127.0.0.1:8989;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_redirect off;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $http_connection;
}
```

### NGINX (Subdomain: `sonarr.yourdomain.tld`)

```nginx
server {
  listen      80;
  listen [::]:80;
  server_name sonarr.*;

  location / {
    proxy_set_header   Host $host;
    proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header   X-Forwarded-Host $host;
    proxy_set_header   X-Forwarded-Proto $scheme;
    proxy_set_header   Upgrade $http_upgrade;
    proxy_set_header   Connection $http_connection;
    proxy_redirect     off;
    proxy_http_version 1.1;
    proxy_pass http://127.0.0.1:8989;
  }
}
```

### Apache (Subdirectory: `/sonarr`)

```apache
<Location /sonarr>
  ProxyPreserveHost on
  ProxyPass http://127.0.0.1:8989/sonarr connectiontimeout=5 timeout=300
  ProxyPassReverse http://127.0.0.1:8989/sonarr
</Location>
```

### Apache (Root VirtualHost)

```apache
ProxyPass / http://127.0.0.1:8989/sonarr/
ProxyPassReverse / http://127.0.0.1:8989/sonarr/
```

> ⚠️ Exclude `/sonarr/api/` from auth middleware if securing via Apache.

</details>

---

## <img src="/truenas.png" class="tab-icon"> TrueNAS Community Edition

> **Use the official TrueNAS app with custom host paths.**

| Step | Action |
|------|--------|
| **1** | **Apps → Discover Apps → Sonarr → Install** |
| **2** | **Port Number → 8989** |
| **3** | **Sonarr Config Storage → Host Path** → `/mnt/tank/configs/sonarr` |
| **4** | **Additional Storage → Host Path** → mount dataset `/mnt/tank/media` → `/media` |
| **5** | Click **Save → Deploy** |

---

# 2 · First‑Run Configuration

> **Set folders, connect a downloader, and add indexers. Done in ~5 minutes.**

## 2.1 Root Folder  <span class="chip">Mandatory</span>

1. **Settings → Media Management → Add Root Folder**
2. Choose **/media/tv** and ensure the switch is set to **Monitored** (green ✔️).

> *If it’s Unmonitored, Sonarr will ignore new episodes!* {.is-info}

## 2.2 Download Client  <span class="chip">qBittorrent</span>

1. **Settings → Download Client → ➕ → qBittorrent**
2. Fill the form:

| Field                 | Example        |
| --------------------- | -------------- |
| Host                  | `10.251.0.244` |
| Port                  | `10095`        |
| Username              | `admin`        |
| Password              | ••••••••       |
| Category              | `tv-sonarr`    |
| Recent/Older Priority | **Last**       |
| Remove Completed      | ✅              |

> **Remote downloader?** Use the **Path Translation** section (bottom of the Download Client page) to map `/downloads` inside qBittorrent to `/media` inside Sonarr.

## 2.3 Indexers (via Prowlarr)

1. Install **[Prowlarr](/Prowlarr)** and connect it to Sonarr (`Settings → Apps → +`).
2. Add indexers in Prowlarr (Jackett, Torznab, etc.).
3. Click **Test → Save** — Sonarr now inherits all indexers automatically.

---

# 3 · Advanced Tweaks *(optional)*

> For users running Recyclarr or tuning quality control.

### Media‑Management Presets

| Field                | Recommended                                                                                                 |
| -------------------- | ----------------------------------------------------------------------------------------------------------- |
| Rename Episodes      | `True`                                                                                                      |
| Episode Formats      | [TRaSH template strings](https://trash-guides.info/Sonarr/Sonarr-recommended-naming-scheme/#episode-format) |
| Series Folder Format | `{Series TitleYear} [imdbid-{ImdbId}]`                                                                      |
| Propers & Repacks    | `Do Not Prefer`                                                                                             |
| Set Permissions      | `True` *(chmod 777)*                                                                                        |

<details><summary><strong>📁 Common Tags / Custom Formats</strong></summary>

| Tag         | Purpose                   |
| ----------- | ------------------------- |
| x265 / HEVC | Prefer modern video codec |
| HDR10 / DV  | Force HDR releases        |
| Atmos       | Require Dolby Atmos audio |
| Anime       | Anime-specific profiles   |

</details>

### Profiles & Quality

Delete default profiles → keep Recyclarr-generated profiles → set Jellyseerr default.

### Metadata & Backups

Enable **Kodi/Emby** metadata.  
Backups: `/media`, **Interval = 1 day**, **Retention = 7**.

<details><summary><strong>🔄 Restoring a Backup</strong></summary>

| Step  | Action                                                                                           |
| ----- | ------------------------------------------------------------------------------------------------ |
| **1** | Stop the Sonarr container / chart                                                                |
| **2** | Copy the latest `*.zip` from `/media/Backups` to your config folder (`/mnt/tank/configs/sonarr`) |
| **3** | In Sonarr: **System → Backup → Restore** → choose the file you just copied                       |
| **4** | Restart Sonarr when prompted and verify your settings/series are back                            |

</details>

---

# 4 · Troubleshooting

> **Start with the Health tab** — Sonarr flags missing root paths, failed downloads, and indexer issues. {.is-info}

<details><summary><strong>Sonarr cannot see media files</strong></summary>

```bash
ls -lah /mnt/tank/media/tv
chown -R 568:568 /mnt/tank/media/tv
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

---

## ✏️ Editors & Contributors

Thank you to everyone who helped improve this guide:

* **Scar13t** — Page Layout & Design

> Want to help? Open a pull request or ping us on Discord!

---

# 📀 5 · Video Guide

[![Promo](/2025-03-24-advanced-media-management-with-s-promo-card.png)](https://www.patreon.com/posts/advanced-media-124639393)

[⇧ Back to top](#what-is-sonarr){.back-top}

---

· Last updated: 07-13-2025