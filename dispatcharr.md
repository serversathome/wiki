---
title: Dispatcharr
description: A guide to deploying Dispatcharr
published: true
date: 2026-07-13T12:29:53.199Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:16.637Z
---

# <img src="/dispatcharr.png" class="tab-icon"> What is Dispatcharr?

Dispatcharr is an open-source command center for managing IPTV streams, EPG data, and VOD content. Think of it as the *arr family's IPTV cousin: it lets you consolidate multiple IPTV providers into a single interface, filter and organize thousands of channels, match or generate EPG guides, and hand the result off to your media server.

It can emulate an HDHomeRun device so Plex, Emby, or Jellyfin discover it as a live TV source and record straight to their own DVR libraries. It also proxies and relays streams in real time, supports FFmpeg transcoding through output profiles, offers multi-user access control, and can be extended with a plugin system. Output can be served as M3U, XMLTV EPG, Xtream Codes API, or an HDHomeRun device.
# 1 · Deploy Dispatcharr
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  dispatcharr:
    image: ghcr.io/dispatcharr/dispatcharr:latest
    container_name: dispatcharr
    restart: unless-stopped
    ports:
      - 9191:9191
    volumes:
      - /mnt/tank/configs/dispatcharr/data:/data
    environment:
      - DISPATCHARR_ENV=aio
      - REDIS_HOST=localhost
      - CELERY_BROKER_URL=redis://localhost:6379/0
      - DISPATCHARR_LOG_LEVEL=info
```

## <img src="/docker.png" class="tab-icon"> Docker + VPN

```yaml
services:
  dispatcharr:
    image: ghcr.io/dispatcharr/dispatcharr:latest
    container_name: dispatcharr
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/dispatcharr/data:/data
    environment:
      - DISPATCHARR_ENV=aio
      - REDIS_HOST=localhost
      - CELERY_BROKER_URL=redis://localhost:6379/0
      - DISPATCHARR_LOG_LEVEL=info
    network_mode: "service:gluetun"

  gluetun:
    image: qmcgaw/gluetun
    container_name: gluetun
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    environment:
      - VPN_SERVICE_PROVIDER=airvpn
      - VPN_TYPE=wireguard
      - WIREGUARD_PRIVATE_KEY=
      - WIREGUARD_PRESHARED_KEY=     #Optional depending on provider/config
      - WIREGUARD_ADDRESSES=
      - SERVER_COUNTRIES=     #Optional depending on provider/config
    ports:
      - 9191:9191
    restart: unless-stopped
```

> 
> Dispatcharr now auto-generates and persists its own secret key inside the `/data` volume, so you no longer need to set `DJANGO_SECRET_KEY` manually. The compose above runs in all-in-one (AIO) mode with Redis and PostgreSQL bundled inside the single container, which is what most homelabbers want. If you need external Postgres/Redis or TLS-encrypted database connections, there is an optional modular deployment documented in the official Dispatcharr docs.
{.is-info}

# 2 · Logging In

1. Navigate to `http://IP:9191` and set a username and password

# 3 · Adding Channels

1. Click the **+ Add M3U** button
2. Give it a name
3. Use a URL for a m3u list like `https://iptv-org.github.io/iptv/languages/eng.m3u`
4. Set the **Account Type** to `Standard`
5. Click the **Save** button at the bottom
6. Click **Configure Groups** in the message window in the bottom right
7. Click the **Groups** button
8. Click **Save and Refresh**
9. Select all Streams in the Dashboard and click **Create Channels** to turn them into live TV channels

# 4 · Linking With Your Media Server

## 4.1 Emby

1. Navigate to the **Admin Panel** by clicking the cog wheel in the upper right corner
2. Click **Live TV**
3. Click **+ Add TV Source**
4. Select **M3U**
5. Get the URL from the purple **M3U** button in the **Channels** tab of Dispatcharr
6. Select the options **Import guide directly from the m3u, when available** and **Allow mapping to guide data using channel numbers**
7. Click **Save**

> Each client will need to make sure they have their dashboard set to show Live TV channels
{.is-info}


## 4.2 Jellyfin

1. Navigate to the **Admin Panel** by clicking the icon of a man in the upper right corner
2. Click **Dashboard**
3. Click **Live TV**
4. Click the **+** next to **Tuner Devices** at the top
5. Under **Tuner Type** select `M3U Tuner`
6. Get the URL from the purple **M3U** button in the **Channels** tab of Dispatcharr
7. Click **Save**

## 4.3 Plex

Plex does not accept a raw M3U tuner, but Dispatcharr's HDHomeRun emulation lets Plex discover it as a network tuner and record to the Plex DVR.

1. In Plex, go to **Settings** > **Live TV & DVR**
2. Click **Set Up Plex DVR**
3. Plex scans your network for tuners and should detect the Dispatcharr HDHomeRun device automatically. If it does not, choose the option to enter the tuner address manually and paste the URL from the **HDHomeRun** button in the **Channels** tab of Dispatcharr
4. Continue, and when prompted for guide data choose to use an **XMLTV** guide, using the URL from the **EPG** button in Dispatcharr (or map to Plex's own guide data)
5. Finish the setup and Plex will list your Dispatcharr channels under Live TV

> 
> I do not run Plex myself, so if you have this working and can tighten up any of the steps above, please edit this page.
{.is-info}

# 5 · Source Lists

The lists below are free, publicly distributed, and actively maintained. They carry the free-to-air streaming feeds (ABC News Live, NBC News NOW, CBS News, LiveNOW from FOX) and the free FAST platforms (Samsung TV Plus, Pluto TV) with their sports, news, and weather channels.

> Lists that include live cable networks like ESPN, FS1, TNT, or "CBS Sports HQ live games" are unstable, and they get taken down frequently causing breakage.
{.is-warning}

## 5.1 M3U Playlists

| Source | Playlist URL | Notes |
|--------|-------------|-------|
| Free-TV USA | `https://cdn.jsdelivr.net/gh/Free-TV/IPTV@master/playlists/playlist_usa.m3u8` | Curated free-to-air. Good primary for the ABC/CBS/NBC/Fox news feeds. |
| iptv-org US | `https://iptv-org.github.io/iptv/countries/us.m3u` | Large community list. Filter to just News, Sports, Weather on import. |
| Samsung TV Plus US | `https://cdn.jsdelivr.net/gh/BuddyChewChew/app-m3u-generator@main/playlists/samsungtvplus_us.m3u` | FAST channels. Most stable. Pairs with a matching guide below. |
| Pluto TV US | `https://cdn.jsdelivr.net/gh/BuddyChewChew/app-m3u-generator@main/playlists/plutotv_us.m3u` | FAST channels. Adds the NBA Channel, UEFA Champions League, and more. |

> Set **Account Type** to `Standard` for all of these, not Xtream Codes.
{.is-info}

## 5.2 EPG Guides

| Source | Guide URL | Notes |
|--------|----------|-------|
| Samsung TV Plus US | `https://i.mjh.nz/SamsungTVPlus/us.xml.gz` | Channel IDs match the Samsung playlist, so **Auto Match** works. |
| Pluto TV US | `https://i.mjh.nz/PlutoTV/us.xml.gz` | Channel IDs match the Pluto playlist. |

> The free network news feeds (ABC/CBS/NBC/Fox) from Free-TV and iptv-org often have thin or missing guide data, since they are 24/7 rolling feeds. Leaving those channels on the Dummy guide is normal and they still play fine. The Samsung and Pluto channels are the ones that populate a full program grid.
{.is-warning}

## 5.3 If a GitHub Link Throws an Error

Raw GitHub links (`raw.githubusercontent.com`) can hit a `429 Too Many Requests` rate limit. When that happens, swap to the **jsDelivr** mirror of the same file, which is a CDN and does not throttle:

```
Raw:      https://raw.githubusercontent.com/USER/REPO/BRANCH/path/file.m3u
jsDelivr: https://cdn.jsdelivr.net/gh/USER/REPO@BRANCH/path/file.m3u
```

> These URLs occasionally move as the upstream repos reorganize. If one returns a 404, open the source repo (Free-TV/IPTV, iptv-org/iptv, BuddyChewChew/app-m3u-generator, matthuisman/i.mjh.nz) and grab the current path. A quick browser test of any URL before adding it will save you a lot of guessing.
{.is-info}