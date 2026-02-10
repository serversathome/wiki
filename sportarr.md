---
title: Sportarr
description: A guide to deploying Sportarr
published: true
date: 2026-02-10T16:32:45.177Z
tags: 
editor: markdown
dateCreated: 2026-02-10T16:23:09.485Z
---

# <img src="/sportarr-light.png" class="tab-icon"> What is Sportarr?

**Sportarr** is a sports PVR (Personal Video Recorder) for Usenet and BitTorrent users. If you've ever used Sonarr or Radarr, you'll feel right at home — Sportarr does the same thing, but for sports events. It monitors sports leagues and events, searches your indexers for releases, and handles file renaming, organization, and metadata integration with Plex, Jellyfin, and Emby.

Sportarr supports **400+ sports leagues** worldwide, including combat sports (UFC, MMA, boxing), American football (NFL), basketball (NBA), soccer (Premier League), hockey (NHL), motorsports (Formula 1), tennis, golf, and many more. It uses a TV show-style naming convention that works seamlessly with media servers.

Key features include Prowlarr integration (Newznab and Torznab API support), IPTV DVR recording (alpha), Plex/Jellyfin/Emby metadata agents, and support for popular download clients like qBittorrent, Transmission, Deluge, SABnzbd, NZBGet, and Decypharr.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Sportarr

```yaml
services:
  sportarr:
    image: sportarr/sportarr:latest
    container_name: sportarr
    environment:
      - PUID=568
      - PGID=568
      - UMASK=022
      - TZ=America/New_York
    ports:
      - "1867:1867"
    volumes:
      - /mnt/tank/configs/sportarr:/config
      - /mnt/tank/media/:/data
    restart: unless-stopped
```

 
> In addition to your `movies` and `tv` folder in your `media` directory, create an additional directory called `sports` with the same permissions
{.is-warning}

# 2 · Configuration

## 2.1 Initial Setup

After deployment, navigate to `http://your-server-ip:1867` and configure the following:

**Root Folder** — Go to **Settings → Media Management** and add a root folder. This is where Sportarr will store your sports library. If you followed the compose above, set this to `/data/sports`.

**Download Client** — Go to **Settings → Download Clients** and add your download client.

**Add Content** — Use the search to find leagues or events. Add them to your library and Sportarr will start monitoring for new releases.

## 2.2 Prowlarr Integration

If you use [Prowlarr](/Prowlarr), you can sync your indexers automatically. Sportarr isn't listed as its own app in Prowlarr yet, but the **Sonarr** option works since Sportarr exposes a compatible API:

1. In Sportarr, go to **Settings → General** and copy your **API Key**
2. In Prowlarr, go to **Settings → Apps → Add Application**
3. Select **Sonarr** as the application type
4. Set the Prowlarr Server and Sportarr Server URLs to `http://sportarr:1867` (or your actual IP/hostname)
5. Paste your Sportarr API key
6. Select **TV (5000)** categories for sync — this includes TV/HD (5040), TV/UHD (5045), and TV/Sport (5060)
7. Click **Test**, then **Save**
8. Indexers will sync automatically and stay updated

## 2.3 Media Server Integration

Sportarr supports metadata integration with Plex, Jellyfin, and Emby so your sports library shows up with proper metadata, artwork, and organization.

**Plex (1.43.0+)** — Use the new Custom Metadata Provider system. No plugin installation required. Go to Plex Web → Settings → Metadata Agents and configure from there. For older Plex versions, download the legacy agent bundle from **Settings → General → Media Server Agents** in the Sportarr UI.

**Jellyfin / Emby** — Configure using the metadata API. See the [Sportarr GitHub](https://github.com/Sportarr/Sportarr) for detailed setup instructions under the `agents/` directory.

Media server connections for library refresh can be configured under the **Connect** page in Sportarr (similar to how Sonarr/Radarr handles it).

> 
> Plex has announced legacy agents will be deprecated in 2026. If you're on Plex 1.43.0+, use the new Custom Metadata Provider method instead.
{.is-warning}

## 2.4 IPTV DVR (Alpha)

Sportarr includes an experimental IPTV DVR feature that can automatically record live sports events.

Features include automatic DVR scheduling, FFmpeg-based IPTV stream recording, auto-import of completed recordings, a TV guide with EPG grid, and filtered M3U/EPG export for external IPTV apps.

**Requirements**: FFmpeg must be installed and accessible in your system PATH, plus a working IPTV source (M3U playlist or Xtream Codes credentials).

Configure IPTV sources under **Settings → IPTV Sources** and DVR settings under **Settings → DVR Recordings**.

> 
> ⚠️ The IPTV DVR functionality is in very early alpha. Expect bugs, missing features, and limited functionality. Use at your own risk and report issues on GitHub.
{.is-danger}


# <img src="/patreon-light.png" class="tab-icon"> 3 · Video

*Coming soon — check back for a full deployment walkthrough!*