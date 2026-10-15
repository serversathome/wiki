---
title: Folder Structure
description: Recommended folder structure to allow for hardlinks within the arr stack
published: true
date: 2026-10-15T18:57:32.122Z
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
    -   **crossseed** – Where Qui places its cross-seed links (reflinks on ZFS), so cross-seeded torrents never duplicate data.

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
   └── dozzle
```

# 2 · Auto Folder Creation and Setup for TrueNAS

I have written a script which creates the datasets necessary for the \*arr stack, builds a docker compose yaml with all apps configured for use with those datasets, launches the stack, and then **configures every app for you** through its API, in the order from the [Install Instructions](/InstallInstructions). All permissions are configured to work out-of-the-box. The script allows split pool installation, for example, if your apps are on an SSD pool and your media is on a HDD pool.

The two datasets it will create are:
1. **configs**
1. **media**

`configs` will have **subdatasets** for each app the script installs (`prowlarr`, `radarr`, `sonarr`, `seerr`, `profilarr`, `bazarr`, `jellyfin`, `qbittorrent`, `qui`, `dozzle`).

`media` has the **subdirectories** `movies`, `tv`, `downloads`, `downloads/torrent` and `downloads/crossseed` instead of datasets for proper hardlinking.

All permissions are set to the following:

| Parameter | Value |
| --- | --- |
| **User ID (UID)** | root (0) |
| **Group ID (GID)** | apps (568) |

> **Permissions**
The script sets `770` POSIX permissions so that the owner and group have full access (r/w/x), while “others” have no access. This helps avoid permissions errors while keeping things secure. The one exception is `configs/seerr`, which is owned by UID 1000 because the Seerr container always runs as that user.
{.is-info}

## 2.1 Running the Script
To view the script, [look here](https://raw.githubusercontent.com/serversathome/ServersatHome/refs/heads/main/truenas-file-structure.sh). 
To run the script, execute this one-liner in the TrueNAS shell:

```bash
sudo su -c "wget https://raw.githubusercontent.com/serversathome/ServersatHome/refs/heads/main/truenas-file-structure.sh && chmod +x truenas-file-structure.sh && bash truenas-file-structure.sh"
```

The script lists the pools on your system and asks you to pick one for the `configs` dataset and one for the `media` dataset (press Enter to use the same pool for both). If you only have one data pool it is used automatically.

> The script is safe to re-run. Datasets and directories that already exist are left in place (permissions are re-applied), an existing compose file is backed up before it is regenerated, and every configuration step reports "already present" instead of doing its work twice.
{.is-info}

## 2.2 Docker Compose File
The custom docker compose file generated for your system will be in `/mnt/POOLNAME/docker/docker-compose.yml`. To view it execute
```bash
cat /mnt/POOLNAME/docker/docker-compose.yml
```
## 2.3 Automated Deployment and Configuration
At the end of the folder setup the script asks whether to **launch and auto-configure** the containers. If you select `yes`:

1. You will be prompted to paste your WireGuard VPN config so qBittorrent can launch immediately with no user intervention (skipped if `configs/qbittorrent/wireguard/wg0.conf` already exists).
1. The script runs `docker compose up -d` to launch all the containers.
1. It then configures each app over its API, waiting for each one to come up, in this order:

| App | What the script sets up |
| --- | --- |
| **qBittorrent** | WebUI login, save paths (`/media/downloads`, incomplete and .torrent files in `/media/downloads/torrent`), `radarr` and `sonarr` categories, connection limits off, torrent queueing off, network interface `wg0` bound to all IPv4 addresses |
| **Radarr / Sonarr** | Forms login, root folders `/media/movies` and `/media/tv`, qBittorrent as download client (by container name, "Remove Completed" off), anonymous usage data off |
| **FlareSolverr** | Checked that it is up (nothing to configure) |
| **Prowlarr** | Forms login, FlareSolverr indexer proxy (tag `flaresolverr`), Radarr and Sonarr apps on full sync, anonymous usage data off |
| **Profilarr** | Account, Radarr and Sonarr linked, all Dictionarry quality profiles plus the delay profile, naming, quality definitions and media settings synced to both (daily at 04:00), drift detection, renames and cleanup enabled, first cleanup run queued |
| **Jellyfin** | Setup wizard completed, admin user, `Movies` and `TV Shows` libraries |
| **Seerr** | Signed in with the Jellyfin admin, libraries enabled, Radarr and Sonarr added with the `1080p Balanced` profile |
| **Bazarr** | Connected to Radarr and Sonarr, anonymous usage data off |
| **Qui** | Account, qBittorrent instance with reflink cross-seed mode (`/media/downloads/crossseed`) and reannounce, cross-seed rules and RSS automation, auto-search on completion, the wiki automations enabled, orphan scan, Radarr and Sonarr integrations, private indexers imported from Prowlarr when present |

4. Finally it prints the URL of every running container, the logins it created, and a **Finish by hand** list for anything it could not complete.

> **Logins created by the script**
> Every app is set up with user `admin`, password `changeme` and (where needed) email `admin@admin.com`. **Change these before exposing anything beyond your LAN.**
{.is-warning}

A few things stay manual on purpose: adding your trackers in Prowlarr (tag Cloudflare-protected ones `flaresolverr`), Bazarr's language profile and subtitle providers, and the one-time Qui library scan once you have private indexers. Re-run the script after adding trackers and it imports your private indexers into Qui.

> If you get the error `"Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?"` navigate to the **Apps** tab and **enable the Apps service**
{.is-warning}

## 2.4 Converting to Dockge (or Portainer, Arcane, TrueNAS)

If you would like to use a container management UI as a front-end manager for this script instead of CLI, run the script in its entirety (selecting `yes` to launch and configure the containers), then run this command in the shell (assuming your pool is named `tank`):
```bash
cd /mnt/tank/docker && docker compose down && cat docker-compose.yml
```

This will stop all containers and output the docker compose file for you to paste as a new stack into Dockge. Paste the compose file and launch the containers and they should instantly start, now being managed within Dockge. All the configuration the script did lives in the `configs` datasets, so nothing is lost by the move.

## <img src="/youtube.png" class="tab-icon"> 2.5 Script Video
https://youtu.be/ho4NUoyU6tM

> If you are completely new to Docker or container volumes, you may want to check out a beginner-friendly guide (e.g. [Docker docs](https://docs.docker.com/get-started/)) before running this setup.
{.is-info}

# <img src="/youtube.png" class="tab-icon"> 3 ·  Hardlinks
https://youtu.be/dD1u0KOWizw


