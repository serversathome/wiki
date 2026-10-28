---
title: Folder Structure
description: Recommended folder structure to allow for hardlinks within the arr stack
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:02:29.149Z
---

 > Having a tough time? [Book time with me](https://links.serversatho.me/hire) 1-on-1 for some help!
{.is-success}
# Arr Stack Folder Structure

This page provides a structured way to organize folders for media management when using the \*arr stack (Radarr, Sonarr, Prowlarr, etc.), ensuring no data duplication. This is based on [Trash Guides](https://trash-guides.info/)' recommendations, but slightly modified.

> **Note on Hardlinks**
> By using subdirectories for **movies**, **tv**, and **downloads**, you allow the \*arr applications to create **hardlinks** instead of copying media files, which helps save disk space and prevents data duplication.
>
>To test if your hardlinks are working, visit [Trash Guides](https://trash-guides.info/File-and-Folder-Structure/Check-if-hardlinks-are-working/).
{.is-info}

The primary dataset is named `media`, and contains three main subdirectories:

-   **movies** – Stores all movie files
-   **tv** – Stores all TV shows
-   **downloads** – Houses downloaded torrents
    -   **torrent** – Stores incomplete torrent files and .torrent metadata files for restoring a BitTorrent client.
    -   **crossseed** – Where Qui places its cross-seed hardlinks, so cross-seeded torrents never duplicate data.

# 1 · Creating the Datasets and Directories

The `media` dataset can be created in the TrueNAS UI, but the subdirectories **need to be created in the shell after** the `media` dataset has been created with the **apps** permissions preset. If your pool was named `tank` the command for this would be:
```bash
mkdir -p /mnt/tank/media/{movies,tv,downloads/torrent,downloads/crossseed}
```

Once the subdirectories have been created, give them the proper permissions by navigating to the **Datasets** tab in the webUI and **Editing** the permissions for the `media` dataset and **applying them recursively**.

The separate dataset `configs` stores the configuration datasets for various applications we'll be deploying. Examples include application settings for qBittorrent, Radarr, Sonarr, and Prowlarr, etc. `configs` should consist of all datasets created in the TrueNAS UI, one per app. 

> **Naming Rules**
> - [x] No spaces within dataset names
> - [x] Do not use capital letters
<!-- {blockquote:.is-danger} -->

When you are done, it should look like this:

```xml
tank
 └── media
   ├── movies
   ├── tv
   └── downloads
        ├── torrent
        └── crossseed
 └── configs
   ├── prowlarr
   ├── radarr
   ├── sonarr
   ├── seerr
   ├── profilarr
   ├── bazarr
   ├── jellyfin
   ├── qbittorrent
   ├── qui
   ├── dozzle
   ├── tracearr
   ├── trawl
   └── dockge
```

# 2 · Auto Folder Creation and Setup for TrueNAS

I have written a script which creates the datasets necessary for the \*arr stack, writes **one Docker compose stack per container** (the layout [Dockge](/Dockge) manages), launches them, and then **configures every app for you** through its API, in the order from the [Install Instructions](/InstallInstructions). All permissions are configured to work out-of-the-box. The script allows split pool installation, for example, if your apps are on an SSD pool and your media is on a HDD pool.

The three datasets it will create are:
1. **configs**
1. **media**
1. **stacks**

`configs` will have **subdatasets** for each app the script installs (`prowlarr`, `radarr`, `sonarr`, `seerr`, `profilarr`, `bazarr`, `jellyfin`, `qbittorrent`, `qui`, `dozzle`, `tracearr`, `trawl`, `dockge`).

`media` has the **subdirectories** `movies`, `tv`, `downloads`, `downloads/torrent` and `downloads/crossseed` instead of datasets for proper hardlinking.

`stacks` has one **directory** per container (`qbittorrent`, `radarr`, `sonarr`, `trawl`, `prowlarr`, `profilarr`, `jellyfin`, `seerr`, `bazarr`, `qui`, `tracearr`, `dozzle`, `watchtower`, `dockge`), each holding its own `compose.yaml`.

`configs` and `stacks` go on the pool you pick for apps; `media` goes on the pool you pick for media:

```xml
tank
 ├── configs
 │   ├── prowlarr, radarr, sonarr, seerr, profilarr, bazarr
 │   ├── jellyfin, qbittorrent, qui, dozzle, trawl, dockge
 │   └── tracearr
 │        ├── postgres
 │        ├── redis
 │        ├── data
 │        └── backup
 ├── stacks
 │   ├── qbittorrent
 │   │    └── compose.yaml
 │   ├── radarr
 │   │    └── compose.yaml
 │   └── ... (one directory per container)
 └── media
     ├── movies
     ├── tv
     └── downloads
          ├── torrent
          └── crossseed
```

All permissions are set to the following:

| Parameter | Value |
| --- | --- |
| **User ID (UID)** | root (0) |
| **Group ID (GID)** | apps (568) |

> **Permissions**
The script sets `770` POSIX permissions so that the owner and group have full access (r/w/x), while “others” have no access. This helps avoid permissions errors while keeping things secure. The one exception is `configs/seerr`, which is owned by UID 1000 because the Seerr container always runs as that user. Tracearr's `postgres`, `redis` and `data` folders are created once and then left to the container, which owns them for its internal database users. Its `backup` folder (`configs/tracearr/backup`, mounted at `/data/backup`) is created owned by UID 1001 (group `apps`), the image's `tracearr` user, because the container does not take ownership of it.
{.is-info}

## 2.1 Running the Script
To view the script, [look here](https://raw.githubusercontent.com/serversathome/ServersatHome/refs/heads/main/arr-complete.sh). 
To run the script, execute this one-liner in the TrueNAS shell:

```bash
sudo su -c "wget -O arr-complete.sh https://raw.githubusercontent.com/serversathome/ServersatHome/refs/heads/main/arr-complete.sh && chmod +x arr-complete.sh && bash arr-complete.sh"
```



The script asks you a few questions up front:

1. **Pools**: it lists the pools on your system and asks you to pick one for `configs` and `stacks` (your SSD pool if you have one) and one for `media` (press Enter to use the same pool). If you only have one data pool it is used automatically. Pool names must be lowercase with no spaces.
1. **Login**: one username and password that it creates in **every** app. Press Enter to keep the defaults `admin` / `changeme`. The username needs at least 3 characters and the password at least 8. Your answers are saved (root-only) in `configs/arr-complete.env`, so a re-run offers them as the defaults.

It then checks for a **GPU** (NVIDIA, Intel or AMD, dedicated or integrated). If it finds one, it tests what the GPU can actually decode and encode using Jellyfin's own ffmpeg in a throwaway container, passes the GPU into the Jellyfin stack, and later sets up Jellyfin's hardware transcoding to match. 

> The script is safe to re-run. Datasets and directories that already exist are left in place (permissions are re-applied), a `compose.yaml` that changed is backed up (`compose.yaml.bak-<date>`) before it is regenerated, and every configuration step reports "already present" instead of doing its work twice. A step that cannot be completed is added to a **Finish by hand** list and the rest of the script still runs.
{.is-success}

## 2.2 Docker Compose Stacks
Each container gets its own compose file at `/mnt/POOLNAME/stacks/<container>/compose.yaml`. To view one, for example Radarr's, execute
```bash
cat /mnt/POOLNAME/stacks/radarr/compose.yaml
```

Every stack joins a shared Docker network named `arr`, so the containers still reach each other by name (`radarr`, `sonarr`, `qbittorrent`, ...) even though they are separate stacks. If you choose not to launch at the end, you can start a stack by hand with:
```bash
docker network create arr
docker compose -f /mnt/POOLNAME/stacks/<container>/compose.yaml up -d
```

| Container | Port | Notes |
| --- | --- | --- |
| **qBittorrent** | 8080 | hotio image, all traffic through WireGuard (`wg0`) |
| **Radarr** | 7878 | |
| **Sonarr** | 8989 | |
| **Trawl** | 8191 | Replaces FlareSolverr (same API), dashboard at `/dashboard` |
| **Prowlarr** | 9696 | |
| **Profilarr** | 6868 | |
| **Jellyfin** | 8096 | GPU passed through when one is found |
| **Seerr** | 5055 | |
| **Bazarr** | 6767 | |
| **Qui** | 7476 | |
| **Tracearr** | 3000 | All-in-one image with its own database |
| **Dozzle** | 8888 | Container logs |
| **Watchtower** | – | Updates the containers daily at 03:00, skips TrueNAS catalog apps |
| **Dockge** | 5001 | Manages everything in `stacks` |

> If you ran an older version of the script with FlareSolverr, re-running it takes the FlareSolverr stack down, moves its compose file aside and points Prowlarr's proxy at Trawl instead.
{.is-info}

## 2.3 Automated Deployment and Configuration
Once the folders and stacks are written, the script asks whether to **launch and auto-configure** the containers. If you select `yes`:

1. qBittorrent needs a WireGuard config (skipped if `configs/qbittorrent/wireguard/wg0.conf` already exists). You can:
   1. **Paste** a `wg0.conf` from your VPN provider
   1. **Auto-generate** one from [AirVPN](/AirVPN) (needs an API key). This uses `airvpn-wg.sh`, which can also set up alerts and automatic server switching.
   1. **Skip** for now. qBittorrent is then not started, the steps that need it are skipped, and the rest of the stack still goes ahead. Add a `wg0.conf` later and re-run the script to finish qBittorrent.
1. It asks for a **forwarded port** if your VPN provider gave you one (AirVPN: **Client Area > Forwarded Ports**), so qBittorrent listens on it. Press Enter to skip.
1. It creates the `arr` network and runs `docker compose up -d` for every stack, including Dockge.
1. It then configures each app over its API, waiting for each one to come up, in this order:

| App | What the script sets up |
| --- | --- |
| **qBittorrent** | WebUI login, save paths (`/media/downloads`, incomplete and .torrent files in `/media/downloads/torrent`), `radarr` and `sonarr` categories saving to `/media/downloads`, bypass authentication for localhost, connection limits off, torrent queueing off, 3 checking torrents, network interface `wg0` bound to all IPv4 addresses, disk IO type Default |
| **Radarr / Sonarr** | Forms login, root folders `/media/movies` and `/media/tv`, qBittorrent as download client (by container name, "Remove Completed" off), anonymous usage data off |
| **Trawl** | Checked that it is up (nothing to configure) |
| **Prowlarr** | Forms login, Trawl as a FlareSolverr indexer proxy (tag `flaresolverr`), Radarr and Sonarr apps on full sync, anonymous usage data off |
| **Profilarr** | Account, Radarr and Sonarr linked, the Dumpstarr database linked for its `Anime 1080p` profile, all Dictionarry quality profiles plus the delay profile, naming, quality definitions and media settings synced to both (daily at 04:00), drift detection, renames and cleanup enabled, first cleanup run queued, upgrades with the Servers@Home Upgrade Filter (every 6 hours) |
| **Jellyfin** | Setup wizard completed, admin user, `Movies` and `TV Shows` libraries with real-time monitoring, hardware transcoding matching your GPU, and a Jellyfin connection in Radarr and Sonarr so libraries refresh on import |
| **Seerr** | Signed in with the Jellyfin admin, libraries enabled, Radarr and Sonarr added with the `1080p Balanced` profile (Sonarr's anime profile set to `Anime 1080p`) |
| **Bazarr** | Forms login, connected to Radarr and Sonarr, anonymous usage data off |
| **Qui** | Account, qBittorrent instance with hardlink cross-seed mode (`/media/downloads/crossseed`, flat) and reannounce, cross-seed rules (`.cross` suffix) and RSS automation, auto-search on completion, the wiki automations enabled (Remove Unlinked, Unregistered, Stalled, Tag Stalled), orphan scan, Radarr and Sonarr integrations, private indexers imported from Prowlarr when present |
| **Tracearr** | Owner account, Jellyfin connected, Seerr linked for request history |
| **Dockge** | Admin user created and authentication switched off, so the UI opens without a login |

5. Finally it prints the URL of every running container, the logins it created, how Jellyfin is transcoding, and a **Finish by hand** list for anything it could not complete.

> **Logins created by the script**
> Every app is set up with the username and password you chose at the start (by default `admin` / `changeme`) and, where needed, email `admin@admin.com`. Dockge rejects letters-only passwords like the default, so in that case it gets a generated password, printed at the end and saved in `configs/dockge/admin-password.txt`. **If you kept the defaults, change them before exposing anything beyond your LAN.**
{.is-warning}

A few things stay manual on purpose: adding your trackers in Prowlarr (tag Cloudflare-protected ones `flaresolverr`), Bazarr's language profile and subtitle providers, and the one-time Qui library scan (**Cross-Seed > Scan**) once you have private indexers. Re-run the script after adding trackers and it imports your private indexers into Qui.

> If you get the error `"Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?"` navigate to the **Apps** tab and **enable the Apps service**
{.is-warning}

## 2.4 Managing the Stacks with Dockge

There is nothing to convert: the script installs [Dockge](/Dockge) and points it at the `stacks` dataset, so every container already shows up there as its own stack. Open `http://YOUR-IP:5001` to start, stop, update or edit any of them. Dockge sees the `stacks` directory at the same path as the host, so a stack you edit in Dockge and one you run with `docker compose -f /mnt/POOLNAME/stacks/<container>/compose.yaml` are the same thing.

> If you edit a `compose.yaml` by hand (or in Dockge), keep in mind that re-running the script regenerates it. Your edited version is not lost: it is saved next to the new one as `compose.yaml.bak-<date>`.
{.is-info}

## <img src="/patreon-light.png" class="tab-icon"> 2.5 Script Video

Coming November 4th

# <img src="/youtube.png" class="tab-icon"> 3 ·  Hardlinks
https://youtu.be/dD1u0KOWizw


