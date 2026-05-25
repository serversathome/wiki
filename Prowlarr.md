---
title: Prowlarr
description: A guide to installing Prowlarr in TrueNAS Scale as well as docker via compose
published: true
date: 2026-05-25T08:10:04.697Z
tags: media management
editor: markdown
dateCreated: 2026-01-15T15:02:45.163Z
---

# ![](/prowlarr.png){class="tab-icon"} What is Prowlarr?

Prowlarr is an indexer manager/proxy built on the popular arr .net/reactjs base stack to integrate with your various PVR apps. Prowlarr supports both Torrent Trackers and Usenet Indexers. It integrates seamlessly with Sonarr, Radarr, Lidarr, and Readarr offering complete management of your indexers with no per app Indexer setup required.

# 1 · Deploy Prowlarr
# tabs {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/prowlarr:/config
    ports:
      - 9696:9696
    restart: unless-stopped
```

### Permissions & Folder Structure
- **PUID / PGID**: Ensure you use a user/group with the correct permissions for accessing media folders. TrueNAS SCALE defaults to 568:568 for apps.
- **Volumes**: The container structure follows a common-sense naming convention, storing configurations under /mnt/tank/configs/prowlarr
- 📌 Refer to the [Folder-Structure](/Folder-Structure) guide for more details.

## <img src="/truenas.png" class="tab-icon"> TrueNAS


- Install Prowlarr from the TrueNAS Community Apps catalog.
- Use the Community version when available.
- Change the **Config Storage Type** to **Host Path** as per the [Folder-Structure](/Folder-Structure) guide.

# 2 · Prowlarr Configuration

## 2.1 Indexers

This is where you will add public or private trackers you are a part of. In the top left corner you will see a "➕ Add Indexer" button. Click it and search for the name of the tracker you want to add. When you click the name of the tracker a form will open to enter the specific info of the tracker.

## 2.2 Linking to Other \*arr's

In the menu on the left, navigate to **Settings** > **Apps** and click the ➕ icon. Select the app you would like to link. Leave all the options set to their defaults except the **Prowlarr Server** and **{App Name} Server**. The server line needs to be the IP and port Prowlarr and the IP and port of the app you want to link. The **API Key** is copied from the app itself (in the app, navigate to **Settings** > **General** and copy the API Key from the Security section). Click **Test** then **Save**.


## 2.3 Flaresolverr

Usually this is only necessary if you use private trackers, but it won't hurt to set it up. Navigate to **Settings** > **Indexers** and then click the "**+**" box. Click the box for **Flaresolverr**. Enter **flaresolverr** for the **Tags** line and the IP of the server and use port 8191 for the **Host** line.

To apply Flaresolverr to an indexer which utilizes Cloudflare, when adding/editing the indexer, enter **flaresolverr** in the **Tags** line.


## 2.4 Notifications

See [this section](https://wiki.serversatho.me/en/Notifications#radarrsonarrprowlarr) of the [Notifications](https://wiki.serversatho.me/Notifications) page.

## 2.5 General > Backups

1. Navigate to the bottom to the **Backups** section
1. Change the folder to `/media`
1. Set the **Interval** to `1` and the **Retention** to your preference (default: 7 days).

# 3 · Routing Indexers Through a VPN

Prowlarr supports per-indexer proxying via **Indexer Proxies + Tags**. This lets you route specific indexers through a VPN while everything else continues to route directly — surgical control without tunneling your entire stack.

```yaml
services:
  gluetun:
    image: qmcgaw/gluetun:latest
    container_name: gluetun
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    environment:
      - VPN_SERVICE_PROVIDER=
      - VPN_TYPE=wireguard
      - WIREGUARD_PRIVATE_KEY=
      - WIREGUARD_PRESHARED_KEY=
      - WIREGUARD_ADDRESSES=
      - SERVER_COUNTRIES=
      - HTTPPROXY=on
      - HTTPPROXY_LOG=on
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/gluetun:/gluetun
    restart: unless-stopped
 
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/prowlarr:/config
    ports:
      - 9696:9696
    restart: unless-stopped
```


> Gluetun's HTTP proxy port (`8888`) is intentionally not exposed on the host — Prowlarr reaches it over the internal network at `http://gluetun:8888`. If you need to access the proxy from outside Docker, add `- 8888:8888/tcp` to Gluetun's `ports:` block.
{.is-info}



## 3.2 Add the Indexer Proxy

1. Navigate to **Settings** > **Indexers** and scroll to **Indexer Proxies**
2. Click the ➕ icon and select **Http**
3. Fill in the form:
   - **Name:** `Gluetun`
   - **Tags:** add a new tag like `vpn`
   - **Host:** `gluetun` (the container name on the shared network)
   - **Port:** `8888`
   - Leave Username and Password blank unless you set `HTTPPROXY_USER` / `HTTPPROXY_PASSWORD` in Gluetun
4. Click **Test**, then **Save**


## 3.3 Tag the Indexer

Edit the indexer you want routed through the VPN, add the `vpn` tag, and save. Only indexers carrying that tag will use the proxy — every other indexer continues to route directly through your home IP.

## 3.4 FlareSolverr-Backed Indexers

If the indexer relies on FlareSolverr to bypass Cloudflare, the HTTP proxy approach won't work alone — FlareSolverr itself makes the outbound request, so Prowlarr's proxy setting is bypassed. To route FlareSolverr through Gluetun, use `network_mode` on the FlareSolverr container instead:

```yaml
services:
  flaresolverr:
    image: ghcr.io/flaresolverr/flaresolverr:latest
    container_name: flaresolverr
    network_mode: "service:gluetun"
    depends_on:
      - gluetun
    restart: unless-stopped
    # Note: do NOT declare ports here — declare 8191 on the gluetun service
```

> 
> `network_mode: "service:gluetun"` shares Gluetun's network namespace, which means all traffic from FlareSolverr goes through the VPN — and FlareSolverr loses its own port declarations. Move the `8191` port to Gluetun's `ports:` block.
{.is-warning}

# <img src="/patreon-light.png" class="tab-icon"> 4 · Video Walkthrough

[![](/2025-01-30-complete-guide-to-prowlarr-the--promo-card.png)](https://www.patreon.com/posts/complete-guide-121131077?utm_medium=clipboard_copy&utm_source=copyLink&utm_campaign=postshare_creator&utm_content=join_link)