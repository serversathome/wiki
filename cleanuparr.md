---
title: Cleanuparr
description: A guide to deploying Cleanuparr via docker
published: true
date: 2025-06-30T11:13:13.012Z
tags: 
editor: markdown
dateCreated: 2025-06-28T12:56:06.603Z
---

> New page under construction
{.is-info}

![cleanuparr.png](/cleanuparr.png)

# What is Cleanuparr?

Automated Download Management. Automatically clean up unwanted, stalled, and malicious downloads from your \*arr applications and download clients. Keep your queues clean and your media library safe.

# Installation

> Read the [official documentation](https://cleanuparr.github.io/Cleanuparr/docs/)
{.is-success}


```yaml
services:
  cleanuparr:
    image: ghcr.io/cleanuparr/cleanuparr:latest
    container_name: cleanuparr
    restart: unless-stopped
    ports:
      - 11011:11011
    volumes:
      - /mnt/tank/configs/cleanuparr:/config
      - /mnt/tank/media:/media
    environment:
      - PORT=11011
      - BASE_PATH=
      - PUID=568
      - PGID=568
      - UMASK=022
      - TZ=America/New_York
```

The `BASE_PATH` variable is for reverse proxy setups but can be left blank.

## Permissions & Folder Structure

- **PUID / PGID**: Ensure you use a user/group with the correct permissions for accessing media folders. TrueNAS SCALE defaults to 568:568 for apps.
- **Volumes**: The container structure follows a common-sense naming convention, storing configurations under /mnt/tank/configs/radarr
- 📌 Refer to the [Folder-Structure](/Folder-Structure) guide for more details.


# Cleanuparr Configuration
## Instances
1. Add your \*arr instance by navigating to the correct tab on the left pane
1. Enter your instance name, URL and API key

## Download Clients
1. Add your qbittorrent instance by navigating to the **Download Clients** tab on the left pane
1. Enter your instance name, client type, host (URL), username, and password

## Cleanup
### General Configuration
1. Uncheck the box for **Display Support Banner** for a cleaner dashboard
1. Change the **Certificate Validation** to **Disabled for Local Addresses** 
1. If you are using [Huntarr](/huntarr), leave the **Enable Search** box unchecked
1. Click **Save**

### Queue Cleaner Configuration
1. Check the **Enable Queue Cleaner** box

#### Failed Import Settings
*These settings are up to you, but the below values are what I use*

1. Set **Max Strikes** to `3`
1. Check the box for **Ignore Private**

#### Stalled Download Settings
*These settings are up to you, but the below values are what I use*

1. Set **Max Strikes** to `3`
1. Check the box for **Reset Strikes On Progress**
1. Check the box for **Ignore Private**

#### Download Settings
*These settings are up to you, but the below values are what I use*

1. Set **Max Strikes** to `3`





