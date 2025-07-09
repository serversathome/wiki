---
title: Sonarr
description: A guide to installing Sonarr in TrueNAS Scale as well as docker via compose
published: true
date: 2025-07-09T10:22:14.648Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:32:51.765Z
---

# ![Sonarr logo](/sonarr.png){style="height:1.6rem;vertical-align\:middle"} What is Sonarr?

Sonarr is a PVR for Usenet and BitTorrent users. It watches RSS feeds for new episodes, grabs, sorts and renames them, then upgrades quality when a better release appears.

---

<details class="quickstart" open>
<summary><strong>🚀 Quick‑Start Checklist</strong></summary>

1. **Deploy container** (Docker Compose *or* TrueNAS chart).
2. **Create** `/media/tv` **root folder** in Sonarr.
3. **Add qBittorrent** as Download Client.
4. *(Optional)* Import Recyclarr profiles & advanced cleanup.

</details>

---

# 1 · Deploy Sonarr

# tabs {.tabset}

## <img src="/truenas.png" class="tab-icon"> TrueNAS install]\(/screen\_shot\_2023-12-08\_at\_3.04.39\_pm.png)

1. Install **Sonarr** from *Community Apps* (choose **Community** image).
2. Set **Config Storage Type → Host Path**.
3. Under **Additional Storage** mount your media dataset.

---

# 2 · First‑Run Configuration

## 2.1 Root Folder

1. **Settings → Media Management → Add Root Folder**
2. Choose `/media/tv`.

## 2.2 Download Client

1. **Settings → Download Client → ➕ → qBittorrent**
2. Fill the form:

| Field                   | Example        |
| ----------------------- | -------------- |
| Host                    | `10.251.0.244` |
| Port                    | `10095`        |
| Username                | `admin`        |
| Password                | •••••••••      |
| Category                | `tv-sonarr`    |
| Recent / Older Priority | **Last**       |
| Remove Completed        | ✅              |

> **Tip:** Use a dedicated qBittorrent category (e.g. `tv-sonarr`) to avoid clashing with other downloads.

---

# 3 · Advanced Tweaks *(optional)*

> **Warning** – For Recyclarr users. Enable **Show Advanced** first. {.is-warning}

### Media‑Management Presets

| Field                | Recommended                            |
| -------------------- | -------------------------------------- |
| Rename Episodes      | `True`                                 |
| Episode Formats      | *TRaSH template strings*               |
| Series Folder Format | `{Series TitleYear} [imdbid-{ImdbId}]` |
| Propers & Repacks    | `Do Not Prefer`                        |
| Set Permissions      | `True` *(chmod `777`)*                 |

### Profiles & Quality

Remove default profiles → keep Recyclarr profiles → set Jellyseerr default.

### Metadata & Backups

Enable **Kodi/Emby** metadata.<br>
Backups: `/media`, **Interval = 1 day**, **Retention = 7**.

---

# 4 · Troubleshooting

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

* Verify **Download Client Path Mapping** matches container paths.
* Confirm Sonarr can access the completed‑downloads directory.

</details>

---

# Video Guide

[![Promo](/2025-03-24-advanced-media-management-with-s-promo-card.png)](https://www.patreon.com/posts/advanced-media-124639393)

[⇧ Back to top](#what-is-sonarr){.back-top}