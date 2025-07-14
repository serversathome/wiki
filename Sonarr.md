---
title: Sonarr
description: A guide to installing Sonarr in TrueNAS Scale as well as docker via compose
published: true
date: 2025-07-14T04:11:52.053Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:32:51.765Z
---

# ![Sonarr](/sonarr.png){class="tab-icon"} What is Sonarr?

**Sonarr** is a TV-series PVR for Usenet and BitTorrent users. It monitors RSS feeds for new episodes, grabs, sorts, and renames them, and upgrades quality when better releases appear.

> 📌 Works great with **qBittorrent**, **Prowlarr**, and **Jellyfin/Plex** for fully automated TV downloads.

<details><summary><strong>📘 How Sonarr Finds Episodes</strong></summary>

Sonarr uses RSS feeds to spot newly uploaded releases and compares them to your monitored shows and quality profiles. If a match is found, it grabs the file automatically.

- **RSS interval:** 15–60 minutes
- **Manual search needed for past episodes**
- **Search triggers:** Missing episodes, manual search, API calls, adding a show with search enabled
- **Auto-search:** Only for recently aired episodes or those added in the last 14 days

</details>

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

<details><summary><strong>💡 Hardlinks vs. Copies</strong></summary>

Sonarr uses **hardlinks** when importing media, avoiding duplicate files and saving space. This only works if the source (e.g. completed downloads) and destination (e.g. `/media/tv`) are on the **same dataset** or filesystem.

If hardlinking fails, Sonarr will fallback to copying.

</details>

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

## <img src="/nginx-proxy-manager.png" class="tab-icon"> NGINX Reverse Proxy

> Configure reverse proxy access for Sonarr via NGINX (subdirectory or subdomain).  
Prefer a GUI? See [NGINX Proxy Manager](/nginx) or [Cloudflare Tunnel](/CloudflareTunnels).

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

...
