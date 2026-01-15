---
title: Plex
description: A guide to installing Plex in TrueNAS and via docker compose
published: true
date: 2026-01-15T15:30:52.797Z
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
## <img src="/docker.png" class="tab-icon"> Docker Compose
```yaml
services:
  plex:
    container_name: plex
    image: plexinc/pms-docker
    restart: unless-stopped
	# devices:
      # - /dev/dri:/dev/dri
    #runtime: nvidia
    #deploy:
    #  resources:
    #    reservations:
    #      devices:
    #        - capabilities:
    #            - gpu
    ports:
      - 32400:32400/tcp
      - 8324:8324/tcp
      - 32469:32469/tcp
      - 1900:1900/udp
      - 32410:32410/udp
      - 32412:32412/udp
      - 32413:32413/udp
      - 32414:32414/udp
    environment:
      - TZ=America/New_York
      - PLEX_CLAIM=
      - ADVERTISE_IP=http://10.99.0.191:32400/
      - PLEX_UID=568
      - PLEX_GID=568
      # - NVIDIA_VISIBLE_DEVICES=all
			# - NVIDIA_DRIVER_CAPABILITIES=all
    hostname: plex
    volumes:
      - /mnt/tank/configs/plex:/config
      - /mnt/tank/configs/plex/transcode:/transcode
      - /mnt/tank/media:/data
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
[](https://youtu.be/L8Wn5EPwuYI)

