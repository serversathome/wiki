---
title: Sonarr
description: A guide to installing Sonarr in TrueNAS Scale as well as docker via compose
published: true
date: 2026-06-07T14:40:13.548Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:02:54.282Z
---

# <img src="/sonarr.png" class="tab-icon"> What is Sonarr?
**Sonarr** is a PVR (personal video recorder) for Usenet and BitTorrent users. It automatically monitors multiple RSS feeds and indexers for new and upgraded episodes of your wanted TV series, grabs them through your download client, then renames and organizes them into your library by series, season, and episode. Think of it as the TV-focused sibling of Radarr — part of the broader *arr stack alongside Prowlarr, Radarr, and Bazarr.
Sonarr handles quality profiles, custom formats, automatic upgrades (e.g. replacing a 720p episode when a 1080p release appears), season pass monitoring, and release calendar tracking so it can grab new episodes the moment they air.
> 
> Sonarr pairs best with **Prowlarr** (indexer management) and a download client such as **qBittorrent** or **SABnzbd**. Set those up first for the smoothest experience.
{.is-info}
# 1 · Deploy Sonarr
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker
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
      - "8989:8989"
    restart: unless-stopped
```

## <img src="/truenas.png" class="tab-icon"> TrueNAS
1. Navigate to **Apps** in the TrueNAS UI.
2. Search the **Community** catalog for "Sonarr" and click **Install**.
3. Configure the following settings:
   - **App Name**: `sonarr`
   - **Timezone**: `America/New_York`
   - **User and Group ID**: PUID `568` / PGID `568`
   - **Storage → Config**: Host Path → `/mnt/tank/configs/sonarr`
   - **Storage → Additional Storage**: Host Path `/mnt/tank/media` mounted to `/media`
   - **Web Port**: `8989`
4. Click **Install** and wait for the app to reach the **Running** state.
# 2 · Configuration
## 2.1 Authentication
Before anything else, lock down access:
1. Go to **Settings → General**.
2. Under **Security**, set **Authentication** to `Forms (Login Page)`.
3. Set **Authentication Required** to `Enabled`.
4. Create a username and password, then save. Sonarr will restart.
## 2.2 Connect Your Download Client
1. Go to **Settings → Download Clients** and click the **+** button.
2. Select your client (e.g. **qBittorrent** or **SABnzbd**).
3. Enter the host, port, and credentials.
4. Click **Test**, then **Save**.
## 2.3 Connect Indexers via Prowlarr
Rather than adding indexers manually, let Prowlarr manage them:
1. In **Prowlarr**, go to **Settings → Apps** and add Sonarr.
2. Enter the Sonarr URL (`http://sonarr:8989`) and API key (found in Sonarr under **Settings → General → Security**).
3. Prowlarr will sync all your indexers to Sonarr automatically.
## 2.4 Quality Profiles & Root Folder
1. Go to **Settings → Media Management** and add a **Root Folder** pointing to `/media/tv`.
2. Under **Settings → Profiles**, configure quality profiles to match your storage and bandwidth.
3. For curated custom formats and quality scoring, consider [Profilarr](/profilarr) to manage profiles across Radarr and Sonarr.
# <img src="/patreon-light.png" class="tab-icon"> 3 · Video Guide 

[![](/2025-03-24-advanced-media-management-with-s-promo-card.png)](https://www.patreon.com/posts/advanced-media-124639393)





