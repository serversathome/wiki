---
title: Sonarr
description: A guide to installing Sonarr in TrueNAS Scale as well as docker via compose
published: true
date: 2025-07-14T04:43:16.670Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:32:51.765Z
---

# ![Sonarr](/sonarr.png){class="tab-icon"} What is Sonarr?

**Sonarr** is a TV-series PVR for Usenet and BitTorrent users. It monitors RSS feeds for new episodes, grabs, sorts, and renames them, and upgrades quality when better releases appear.

> 📌 Works great with **qBittorrent**, **Prowlarr**, and **Jellyfin / Plex** for fully automated TV downloads.

---

# 1 · Deploy Sonarr

> **Start by setting up your datasets, then choose your install method.**

# tabs {.tabset}

## 📂 Folder Setup

> Create the following datasets in **TrueNAS** or match these paths in **Docker volumes**.

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
      - 8989:8989 # Use 127.0.0.1:8989 for reverse proxy setups
    restart: unless-stopped
```

> **Behind a reverse‑proxy?** Expose port **8989** on `127.0.0.1` and route through Nginx Proxy Manager or Cloudflare Tunnel.

---

## <img src="/truenas.png" class="tab-icon"> TrueNAS Community Edition

> Use the official TrueNAS Sonarr app with custom host paths.

| Step | Action                                                         |
| ---- | -------------------------------------------------------------- |
| 1    | Apps → Discover Apps → Sonarr → Install                        |
| 2    | Set **Port** to 8989                                           |
| 3    | Sonarr Config Storage → Host Path → `/mnt/tank/configs/sonarr` |
| 4    | Additional Storage → Host Path → `/mnt/tank/media` → `/media`  |
| 5    | Click **Save** → **Deploy**                                    |

---

## <img src="/nginx-proxy-manager.png" class="tab-icon"> NGINX Reverse Proxy

> Configure a reverse proxy (subdirectory or subdomain). Prefer a GUI? See [NGINX Proxy Manager](/nginx) or [Cloudflare Tunnel](/CloudflareTunnels).

### Subdirectory `/sonarr`

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

### Subdomain `sonarr.yourdomain.tld`

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

---

# 2 · First‑Run Configuration

> **Set folders, connect a downloader, and add indexers. Done in \~5 minutes.**

# tabs {.tabset}

## 📁 Library

<details open><summary><strong>⚙️ Media Management</strong></summary>

* ✅ Enable **Rename Episodes**
* Include quality and release group in naming
* Enable **Use Hard Links instead of Copy** for performance
* ➕ Add `.srt` to **Import Extra Files**

</details>

<details><summary><strong>📁 Root Folders</strong></summary>

* Add `/media/tv` as a monitored root folder
* Ensure Sonarr has read/write access

</details>

## 🔍 Indexers

<details open><summary><strong>📡 Configure via Prowlarr</strong></summary>

1. Install **[Prowlarr](/Prowlarr)**
2. In Sonarr: **Settings → Apps → +** and connect Prowlarr
3. Add indexers (Jackett, Torznab, etc.) within Prowlarr
4. **Test → Save** — Sonarr will import these indexers automatically

</details>

## 📥 Download Clients

<details open><summary><strong>qBittorrent Setup</strong></summary>

| Field            | Example        |
| ---------------- | -------------- |
| Host             | `10.251.0.244` |
| Port             | `10095`        |
| Username         | `admin`        |
| Password         | ••••••••       |
| Category         | `tv-sonarr`    |
| Remove Completed | ✅              |

> Remote downloader? Use **Path Translation** to map `/downloads` inside qBittorrent to `/media` inside Sonarr.

</details>

## 🎯 Profiles

<details open><summary><strong>Quality Profiles</strong></summary>

* Create or customize profiles with desired sources
* Remove defaults you don't use
* Set **Cutoff** to stop upgrades when target quality is reached

</details>

## 🛠️ Automation

<details open><summary><strong>Recyclarr Setup (Optional)</strong></summary>

* Sync TRaSH naming templates, profiles, and quality settings
* Useful for multiple libraries (4K vs 1080p)
* Place Recyclarr config in Sonarr's `/config` or a separate folder

</details>

---

# 3 · FAQ

<details><summary><strong>Why didn’t Sonarr grab an episode?</strong></summary>

Check RSS sync timing, run a manual search, and ensure indexers are active.

</details>

<details><summary><strong>What do status colors mean?</strong></summary>

* **Blue**: Download in progress
* **Yellow**: Warning (e.g. missing file)
* **Red**: Error

</details>

<details><summary><strong>How to back up and restore?</strong></summary>

* **Backup**: System → Backup → Download .zip
* **Restore**: System → Backup → Restore → Upload .zip

</details>

---

# 4 · Advanced Tweaks *(optional)*

> For power users tuning profiles, metadata, or running multiple instances.

<details><summary><strong>🧩 Media Management Presets</strong></summary>

| Field           | Recommendation                         |
| --------------- | -------------------------------------- |
| Rename Episodes | True                                   |
| Episode Format  | `{Series TitleYear} [imdbid-{ImdbId}]` |
| Use Hard Links  | True                                   |

</details>

<details><summary><strong>🧬 Multiple Instances</strong></summary>

* Different `/config` folders and external ports
* Unique root folders, download categories, and container names

</details>

---

# 5 · Troubleshooting

> **Start with the Health tab** to spot missing paths or failed downloads.

<details><summary><strong>Permission Denied</strong></summary>

```bash
chmod -R 770 /mnt/tank/media/tv
chown -R 568:568 /mnt/tank/media/tv
```

</details>

<details><summary><strong>Downloads not importing</strong></summary>

Verify path translation and permissions between Sonarr and your client.

</details>

---

## ✏️ Editors & Contributors

* **Scar13t** — Page Layout & Design

> Want to help? Open a PR or join us on Discord!

---

# 📀 6 · Video Guide

[![Watch on Patreon](/2025-03-24-advanced-media-management-with-s-promo-card.png)](https://www.patreon.com/posts/advanced-media-124639393)

[⇗ Back to top](#what-is-sonarr){.back-top}
