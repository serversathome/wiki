---
title: Folder Structure
description: Recommended folder structure from Trash Guides to allow for hardlinks within the arr stack
published: true
date: 2025-07-09T12:21:32.867Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:27:24.035Z
---

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

# 1 · Creating the Datasets and Directories

The `media` dataset can be created in the TrueNAS UI, but the subdirectories **need to be created in the shell after** the `media` dataset has been created with the **apps** permissions preset. If your pool was named `tank` the command for this would be:
```bash
mkdir -p /mnt/tank/media/{movies,tv,downloads}
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
        └── torrent
 └── configs
   ├── prowlarr
   ├── radarr
   ├── sonarr
   ├── jellyseerr
   ├── recyclarr
   ├── bazarr
   ├── tdarr
   ├── jellyfin
   ├── qbittorrent
   └── dozzle
```

# 2 · Auto Folder Creation for TrueNAS

I have written a script which will create the datasets necessary for the \*arr stack as well as build a docker compose yaml with all apps configured for use with those datasets. All permissions are configured to work out-of-the-box.

The two datasets it will create are:
1. **configs**
1. **media**

`media` has the subdirectories `movies`, `tv`, and `downloads` instead of datasets for proper hardlinking.

All permissions are set to the following:

| Parameter | Value |
| --- | --- |
| **User ID (UID)** | root (0) |
| **Group ID (GID)** | apps (568) |

> **Permissions**
The script sets `770` POSIX permissions so that the owner and group have full access (r/w/x), while “others” have no access. This helps avoid permissions errors while keeping things secure.
{.is-info}

## 2.1 Running the Script
To view the script, [look here](https://raw.githubusercontent.com/imjustleaving/ServersatHome/refs/heads/main/truenas-file-structure.sh). 
To run the script, execute this one-liner in the TrueNAS shell:

```bash
sudo su -c "wget https://raw.githubusercontent.com/serversathome/ServersatHome/refs/heads/main/truenas-file-structure.sh && chmod +x truenas-file-structure.sh && bash truenas-file-structure.sh"
```

The script will then ask you to choose the pool to install all the datasets to. 

> Make sure your pool name is all lowercase and has no spaces!
{.is-danger}

> If the script finds any of the paths existing already it will skip it with no modifications.
{.is-info}

## 2.2 Docker Compose File
The custom docker compose file generated for your system will be in `/mnt/POOLNAME/docker/docker-compose.yml`. To view it execute
```bash
cat /mnt/POOLNAME/docker/docker-compose.yml
```
## 2.3 Automated Deployment
At the end of the script it will prompt you would to deploy the containers. If you select `yes`, this will execute a `docker compose up -d` to launch all the containers from the shell. From there the following steps will be executed:

1. You will be prompted to the paste your wireguard VPN key so qBittorrent can launch immediately with no user intervention
1. You will be prompted to sync recyclarr with radarr and sonarr. If you choose this, it will sync the 1080p and 4K profiles found in the [example recyclarr.yml](https://wiki.serversatho.me/en/Recyclarr#recyclarryml) file with radarr and sonarr. 
1. The script will set `/media/downloads` as the default save path for qBittorrent
1. The script will set the root folders for Radarr and Sonarr to the correct paths
1. The script will print out a list of URLs for all running containers as well as the password generated to login to qBittorrent for the first time

> If you get the error `"Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?"` navigate to the **Apps** tab and **enable the Apps service**
{.is-warning}

## 2.4 Converting to Dockge (or Portainer, Arcane, TrueNAS)

If you would like to use a container management UI as a front-end manager for this script instead of CLI, run the script in its entirety (selecting `yes` to launch the containers), then run this commmand in the shell (assuming your pool is named `tank`):
```bash
cd /mnt/tank/docker && docker compose down && cat docker-compose.yml
```

This will stop all containers and output the docker compose file for you to paste as a new stack into Dockge. Paste the compose file and launch the containers and they should instantly start, now being managed within Dockge.

## 2.5 Script Video
[](https://youtu.be/8gATbBJHc5o)

> If you are completely new to Docker or container volumes, you may want to check out a beginner-friendly guide (e.g. [Docker docs](https://docs.docker.com/get-started/)) before running this setup.
{.is-info}

# 3 ·  Hardlinks
[](https://youtu.be/dD1u0KOWizw)

# 4 · Key Takeaways

1.  Media files are stored under `media` and are neatly categorized into movies, tv, and downloads/torrent for torrent handling. For example, the file paths would be:

| Category | Example Path |
| --- | --- |
| Movies | /media/movies/Inception (2010)/Inception.mkv |
| TV Shows | /media/tv/Breaking Bad/Season 1/Breaking Bad S01E01.mkv |

1.  Configuration files are kept in a separate `configs` directory for standard Docker/TrueNAS setups, while Dockge places these inside `stacks`.
2.  TrueNAS users benefit from the `configs` directory while other systems (e.g., Proxmox LXCs, Ubuntu) may have different volume handling.
3.  Dockge users should place all container volumes inside `stacks` for easier backups and migrations.