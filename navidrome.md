---
title: Navidrome
description: A guide to deploying the Navidrome music player using docker
published: true
date: 2025-07-13T22:16:10.415Z
tags: 
editor: markdown
dateCreated: 2025-07-13T10:43:27.715Z
---

# ![Navidrome](/navidrome.png){class="tab-icon"} What is Navidrome?

**Navidrome** is a lightweight self‑hosted music streaming server (Subsonic compatible). It scans your music library, then lets you stream or download tracks via the web‑UI or mobile apps like DSub/Ultrasonic.

---

<details class="quickstart" open>
<summary><strong>🚀 Quick‑Start Checklist</strong></summary>

1. **Deploy container** (Docker Compose *or* TrueNAS App)
2. **Mount music** → `/music`, **Config** → `/data`
3. Wait for **first scan** (progress top‑left)
4. Create **admin user** → set password
5. *(Optional)* Tag library cleanly with **MusicBrainz Picard** or **Beets**

</details>

---

# 1 · Deploy Navidrome

# tabs {.tabset}

## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  navidrome:
    image: deluan/navidrome:latest
    container_name: navidrome
    user: 568:568            # PUID:PGID (TrueNAS default)
    environment:
      ND_SCANNER_SCHEDULE: 1h   # rescans every hour
      ND_LOGLEVEL: info
      ND_SESSIONTIMEOUT: 12h
    volumes:
      - /mnt/tank/configs/navidrome/data:/data
      - /mnt/tank/media/music:/music
    ports:
      - 4533:4533
    restart: unless-stopped
```

### Permissions & Folder Structure {.is-success}

* **PUID / PGID** – use your media‑owner account (`568:568` on SCALE).
* **Volumes** – `/data` stores library cache & artwork, `/music` is read‑only music share.
  📌 See the [Folder‑Structure](/Folder-Structure) guide.

> Official env‑vars list: [Navidrome Docker Docs](https://www.navidrome.org/docs/installation/docker/) {.is-info}

---

## <img src="/truenas.png" class="tab-icon"> TrueNAS Community Edition

| Step  | Action                                                                                      |
| ----- | ------------------------------------------------------------------------------------------- |
| **1** | **Apps → Discover Apps → Navidrome → Install**                                              |
| **2** | **Config Storage → Host Path** → `/mnt/tank/configs/navidrome/data`                         |
| **3** | **Additional Storage → Host Path** → mount `/mnt/tank/media/music` ➜ `/music` *(read‑only)* |
| **4** | Expose port **4533** → 4533                                                                 |
| **5** | **Save → Deploy**                                                                           |

---

# 2 · First‑Run Configuration

1. Browse to `http://SERVER_IP:4533` → create **admin** user.
2. Navidrome scans your library automatically. Watch progress top‑left.
3. **Settings → Transcoding** — enable optional MP3/Opus profiles for mobile data‑savings.
4. **Settings → Last.fm** — add API key if you want scrobbling.

---

# 3 · Recommended Tweaks *(optional)*

| Setting                 | Recommended | Why                                  |
| ----------------------- | ----------- | ------------------------------------ |
| **Scanner Schedule**    | `1h`        | Picks up new music hourly            |
| **Cover Size Limit**    | `6000` px   | Keeps huge artwork but not originals |
| **Transcoding Threads** | `4`         | Speed on multi‑core CPUs             |
| **Session Timeout**     | `12h`       | Stay logged in on home network       |

---

# 4 · Troubleshooting

<details><summary><strong>No music found after scan</strong></summary>
- Confirm path `/music` contains FLAC/MP3 files (not inside artist/album subfolders? That’s okay).  
- Check **Logs → Level INFO** for `scanner` errors (permissions, invalid tags).
</details>

<details><summary><strong>Cover art missing</strong></summary>
- Navidrome looks for `cover.jpg|png` or `folder.jpg` per album.  
- Embed artwork with **MusicBrainz Picard** or `beet embedart`.
</details>

<details><summary><strong>Slow transcoding / buffering</strong></summary>
- Enable **hardware transcoding**: set `ND_TRANSCODING_FFMPEG_PARAMS="-threads 2 -codec:a libmp3lame -b:a 192k"`.  
- Or raise CPU limits on the container.
</details>

---

## ✏️ Editors & Contributors

> **Special thanks to the following members for reviewing and polishing this guide**
> - Dayce
> - Scar13t

Feel free to open a pull‑request or ping us on Discord if you spot an inaccuracy!

---

# <img src="/youtube.png" class="tab-icon"> Video Guide

*Coming soon – subscribe to our [YouTube](https://www.youtube.com/@ServersatHome).*

[⇧ Back to top](#what-is-navidrome){.back-top}
