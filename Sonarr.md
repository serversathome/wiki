---
title: Sonarr
description: A guide to installing Sonarr in TrueNAS Scale as well as docker via compose
published: true
date: 2025-07-09T10:56:31.950Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:32:51.765Z
---

# ![Sonarr](/sonarr.png){class="tab-icon"} What is Sonarr?

Sonarr is a PVR for Usenet and BitTorrent users. It monitors RSS feeds for new episodes, grabs, sorts and renames them, then upgrades quality when a better release appears.

---

<details class="quickstart" open>
<summary><strong>🚀 Quick-Start Checklist</strong></summary>

1. **Deploy container** (Docker Compose *or* TrueNAS chart).
2. **Create** `/media/tv` **root folder** in Sonarr.
3. **Add qBittorrent** as Download Client.
4. *(Optional)* Import Recyclarr profiles & advanced cleanup.

</details>

---

# 1 · Deploy Sonarr

# tabs {.tabset}

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

### Permissions & Folder Structure {.is-success}

* **PUID / PGID** – media-owner UID/GID (TrueNAS SCALE default **568:568**).
* **Volumes** – configs at `/mnt/tank/configs/sonarr`, media at `/mnt/tank/media`.
  📌 See the [Folder-Structure](/Folder-Structure) guide.

---

## <img src="/truenas.png" class="tab-icon"> TrueNAS Community Edition

| Step  | Action                                                                          |
| ----- | ------------------------------------------------------------------------------- |
| **1** | **Apps → Discover Apps → Sonarr → Install**                                     |
| **2** | **Port Number → 8989**                                                          |
| **3** | **Sonarr Config Storage → Host Path** → `/mnt/tank/configs/sonarr`              |
| **4** | **Additional Storage → Host Path** → mount dataset `/mnt/tank/media` ➜ `/media` |
| **5** | Click **Save → Deploy**                                                         |

---

### ▶ Prototype — Two-Column Step Cards (side-by-side)

<div class="step-grid">
  <div class="step-card"><span class="step-num">①</span><br><strong>Pull Image</strong><br><code>docker pull lscr.io/linuxserver/sonarr</code></div>
  <div class="step-card"><span class="step-num">②</span><br><strong>Create Volumes</strong><br><code>mkdir -p /mnt/tank/configs/sonarr</code></div>
  <div class="step-card"><span class="step-num">③</span><br><strong>Generate Compose</strong><br>Paste YAML ↖️</div>
  <div class="step-card"><span class="step-num">④</span><br><strong>Up the stack</strong><br><code>docker compose up -d</code></div>
</div>

> *These “step cards” use simple flexbox (`.step-grid { display:flex; flex-wrap:wrap; gap:1rem }`). They live **under** the existing prose for easy A/B comparison.*

---

# 2 · First-Run Configuration

## 2.1 Root Folder

1. **Settings → Media Management → Add Root Folder**
2. Choose **/media/tv**.

## 2.2 Download Client

1. **Settings → Download Client → ➕ → qBittorrent**
2. Fill the form:

| Field                   | Example        |
| ----------------------- | -------------- |
| Host                    | `10.251.0.244` |
| Port                    | `10095`        |
| Username                | `admin`        |
| Password                | ••••••••       |
| Category                | `tv-sonarr`    |
| Recent / Older Priority | **Last**       |
| Remove Completed        | ✅              |

> **Tip:** A dedicated category (e.g. `tv-sonarr`) keeps Sonarr torrents separate from others.

---

### ▶ Prototype — Icon Tabs for Root vs Download Client

# tabs {.tabset}

#### <img src="/folder-icon.png" class="tab-icon"> Root Folder

* Settings → **Media Management** → **Add Root Folder**.
* Path: **/media/tv** (matches the Docker volume).

#### <img src="/qbittorrent.png" class="tab-icon"> Download Client

Same qBittorrent table as above (duplicated for comparison).

> *These secondary tabs mirror the Docker/TrueNAS style. Decide later which version you prefer, then delete the other.*

---

# 3 · Advanced Tweaks *(optional)*

> **Warning** – For Recyclarr users. Enable **Show Advanced** first. {.is-warning}

### Media-Management Presets

| Field                | Recommended                            |
| -------------------- | -------------------------------------- |
| Rename Episodes      | `True`                                 |
| Episode Formats      | *TRaSH template strings*               |
| Series Folder Format | `{Series TitleYear} [imdbid-{ImdbId}]` |
| Propers & Repacks    | `Do Not Prefer`                        |
| Set Permissions      | `True` *(chmod 777)*                   |

### Profiles & Quality

Delete default profiles → keep Recyclarr-generated profiles → set Jellyseerr default.

### Metadata & Backups

Enable **Kodi/Emby** metadata. Backups: `/media`, **Interval = 1 day**, **Retention = 7**.

---

# 4 · Troubleshooting

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

# Video Guide

[![Promo](/2025-03-24-advanced-media-management-with-s-promo-card.png)](https://www.patreon.com/posts/advanced-media-124639393)

[⇧ Back to top](#what-is-sonarr){.back-top}
