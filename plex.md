---
title: Plex
description: A guide to installing Plex in TrueNAS and via docker compose
published: true
date: 2026-02-11T21:31:22.699Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:21.349Z
---

> Plex has recently made streaming outside your home network a paid-only feature. [Read about it here](https://www.plex.tv/blog/important-2025-plex-updates/). 
{.is-danger}

# <img src="/plex.png" class="tab-icon"> What is Plex?
A one-stop destination to stream movies, TV shows, and music, Plex is the most comprehensive entertainment platform available today. Available on almost any device, Plex is the first-and-only streaming platform to offer free ad-supported movies, shows, and live TV together with the ability to easily search—and add to your Watchlist—any title ever made, no matter which streaming service it lives on. Using the platform as their entertainment concierge, 17 million (and growing!) monthly active users count on Plex for new discoveries and recommendations from all their favorite streaming apps, personal media libraries, and beyond.

# 1 · Deploy Plex
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose Nvidia
```yaml
services:
  plex:
    image: lscr.io/linuxserver/plex:latest
    container_name: plex
    network_mode: host
    runtime: nvidia
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
      - VERSION=docker
      - NVIDIA_VISIBLE_DEVICES=all
      - NVIDIA_DRIVER_CAPABILITIES=compute,video,utility
    volumes:
      - /mnt/tank/configs/plex/configs:/config
      - /mnt/tank/configs/plex/logs:/logs
      - /mnt/tank/configs/plex/transcode:/transcode
      - /mnt/tank/media:/media
    restart: unless-stopped
    # http://{TrueNAS IP}:32400/web/index.html
```
> To get your Plex Claim Token, go to https://plex.tv/claim and follow the steps to create an account (or login to an existing one) to claim your token.
{.is-info}


### Permissions & Folder Structure
- **PLEX_UID / PLEX_GID**: Ensure you use a user/group with the correct permissions for accessing media folders. TrueNAS SCALE defaults to 568:568 for apps.
- **Volumes**: The container structure follows a common-sense naming convention, storing configurations under /mnt/tank/configs/plex
- 📌 Refer to the [Folder-Structure](/Folder-Structure) guide for more details.

> **Enabling nVidia GPUs in Docker**
> Follow the steps [in this Github page](https://github.com/NVIDIA/nvidia-container-toolkit) to allow passing an nVidia GPU into a container
{.is-info}
### GPU Transcode
**NVIDIA**
To enable transcoding from your nVidia GPU, uncomment out the lines in the compose file above.
**Intel iGPU**
To enable transcoding from your Intel iGPU, uncomment out the devices lines (under retstart) in the compose file above.

## <img src="/docker.png" class="tab-icon"> Docker Compose AMD
```yaml
services:
  plex:
    image: lscr.io/linuxserver/plex:latest
    container_name: plex
    network_mode: host
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
      - VERSION=latest
			- DOCKER_MODS=jefflessard/plex-vaapi-amdgpu-mod
      - LIBVA_DRIVERS_PATH=/vaapi-amdgpu/lib/dri 
      - LD_LIBRARY_PATH=/vaapi-amdgpu/lib
      - PLEX_CLAIM= #optional
    group_add:
      - video
      - render
    volumes:
      - /mnt/tank/configs/plex/configs:/config
      - /mnt/tank/configs/plex/logs:/logs
      - /mnt/tank/configs/plex/transcode:/transcode
      - /mnt/tank/media:/media
    devices:
      - /dev/dri:/dev/dri
    restart: unless-stopped
    # http://{TrueNAS IP}:32400/web/index.html
```

> To get your Plex Claim Token, go to https://plex.tv/claim and follow the steps to create an account (or login to an existing one) to claim your token.
{.is-info}

### Permissions & Folder Structure
- **PLEX_UID / PLEX_GID**: Ensure you use a user/group with the correct permissions for accessing media folders. TrueNAS SCALE defaults to 568:568 for apps.
- **Volumes**: The container structure follows a common-sense naming convention, storing configurations under /mnt/tank/configs/plex
- 📌 Refer to the [Folder-Structure](/Folder-Structure) guide for more details.

### Test Hardware Acceleration

To quickly check if hardware acceleration is working, run the following and check for vaapi errors:

```bash
docker exec -it -e LIBVA_DRIVERS_PATH=/vaapi-amdgpu/lib/dri -e LD_LIBRARY_PATH=/vaapi-amdgpu/lib plex \
/lib/plexmediaserver/Plex\ Transcoder -hide_banner -loglevel debug -vaapi_device /dev/dri/renderD128
```

In case the Plex can't access `/dev/dri/renderD128` add `rwx` permissions to owner and owner group:

```bash
truenas_admin@truenas:$ sudo chown -R 770 /dev/dri
```

> **IMPORTANT:** This is not persistent change so after reboot/shutdown you will have to run it `chown` command again!
{.is-warning}

## <img src="/truenas.png" class="tab-icon"> TrueNAS
![screenshot_from_2025-03-13_06-45-49.png](/screenshot_from_2025-03-13_06-45-49.png)

1. Add your **Claim Token**
> To get your Plex Claim Token, go to https://plex.tv/claim and follow the steps to create an account (or login to an existing one) to claim your token.
{.is-info}
2. Change to **Storage Configuration** to the values below: 

| Field | Value | Path |
| --- | --- | --- |
| Plex Data Storage | Host Path | `/mnt/tank/media` |
| Plex Configuration Storage | Host Path | `/mnt/tank/configs/plex/config` |
| Plex Logs Storage | Host Path | `/mnt/tank/configs/plex/logs` |
| Plex Transcode Storage | Temporary or tmpfs | n/a |

3. **GPU Configuration**: Select NVIDIA GPU(s) if you have any available

> Check out [the new docs](https://apps.truenas.com/resources/deploy-plex) from TrueNAS
{.is-success}

# 2 · Plex Configuration
Upon first launch follow these steps:
1. Skip the section for Plex Pass unless you want to buy one
1. Give your server a name and click **Next**
1. Click **Add Library**
1. Select **Movies** and click **Next**
1. Click **Browse for Media Folder**
1. Select **Data** then **Movies** then click **Add**
1. Click **Add Library**
1. Repeat this process for TV Shows
1. Click **Next** then click **Done** then click **Finish Setup**

# <img src="/youtube.png" class="tab-icon"> 3 · Video Walkthrough
https://youtu.be/L8Wn5EPwuYI

