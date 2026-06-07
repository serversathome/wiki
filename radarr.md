---
title: Radarr
description: A guide to installing Radarr in TrueNAS Scale as well as docker via compose
published: true
date: 2026-06-07T14:35:14.113Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:53.605Z
---

# <img src="/radarr.png" class="tab-icon"> What is Radarr?

**Radarr** is a movie collection manager for Usenet and BitTorrent users. It automatically monitors multiple RSS feeds and indexers for new and upgraded releases of your wanted movies, grabs them through your download client, then renames and organizes them into your library. Think of it as the movie-focused sibling of Sonarr — part of the broader *arr stack alongside Prowlarr, Sonarr, and Bazarr.

Radarr handles quality profiles, custom formats, automatic upgrades (e.g. replacing a 720p copy when a 1080p release appears), and list-based imports from sources like Trakt, IMDb, and TMDb.

> 
> Radarr pairs best with **Prowlarr** (indexer management) and a download client such as **qBittorrent** or **SABnzbd**. Set those up first for the smoothest experience.
{.is-info}

# 1 · Deploy Radarr
# {.tabset}
## Docker

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
      - "7878:7878"
    restart: unless-stopped
```

1. Deploy the stack in **Dockge**.
2. Browse to `http://<your-server-ip>:7878` to load the Radarr UI.
3. Set up authentication immediately under **Settings → General** (see section 2.1).

> 
> The `/mnt/tank/media:/data` mount gives Radarr a single unified view of your downloads and movie library. This enables **hardlinks** and **atomic moves** instead of slow copy-and-delete operations — keep downloads and media under the same mount point.
{.is-success}

## TrueNAS

1. Navigate to **Apps** in the TrueNAS UI.
2. Search the **Community** catalog for "Radarr" and click **Install**.
3. Configure the following settings:
   - **App Name**: `radarr`
   - **Timezone**: `America/New_York`
   - **User and Group ID**: PUID `568` / PGID `568`
   - **Storage → Config**: Host Path → `/mnt/tank/configs/radarr`
   - **Storage → Additional Storage**: Host Path `/mnt/tank/media` mounted to `/media`
   - **Web Port**: `7878`
4. Click **Install** and wait for the app to reach the **Running** state.



# 2 · Configuration

## 2.1 Authentication

Before anything else, lock down access:

1. Go to **Settings → General**.
2. Under **Security**, set **Authentication** to `Forms (Login Page)`.
3. Set **Authentication Required** to `Enabled`.
4. Create a username and password, then save. Radarr will restart.


## 2.2 Connect Your Download Client

1. Go to **Settings → Download Clients** and click the **+** button.
2. Select your client (e.g. **qBittorrent** or **SABnzbd**).
3. Enter the host, port, and credentials.
4. Click **Test**, then **Save**.

## 2.3 Connect Indexers via Prowlarr

Rather than adding indexers manually, let Prowlarr manage them:

1. In **Prowlarr**, go to **Settings → Apps** and add Radarr.
2. Enter the Radarr URL (`http://radarr:7878`) and API key (found in Radarr under **Settings → General → Security**).
3. Prowlarr will sync all your indexers to Radarr automatically.

## 2.4 Quality Profiles & Root Folder

1. Go to **Settings → Media Management** and add a **Root Folder** pointing to `/data/media/movies`.
2. Under **Settings → Profiles**, configure quality profiles to match your storage and bandwidth.
3. For curated custom formats and quality scoring, consider **Profilarr** to manage profiles across Radarr and Sonarr.




# <img src="/patreon-light.png" class="tab-icon"> 3 · Video Guide 

[![Promo](/2025-03-18-advanced-media-management-with-r-promo-card.png)](https://www.patreon.com/posts/advanced-media-124637606)
