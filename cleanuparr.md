---
title: Cleanuparr
description: A guide to deploying Cleanuparr via docker
published: true
date: 2025-06-30T11:31:47.758Z
tags: 
editor: markdown
dateCreated: 2025-06-28T12:56:06.603Z
---

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
> If at any time you do not know what a setting does, click the ⓘ symbol next to it to be taken to the official document page explaining it
{.is-info}

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

#### Slow Download Settings
*These settings are up to you, but the below values are what I use*

1. Set **Max Strikes** to `3`
1. Check the box for **Ignore Private**
1. Set **Minimum Speed** to `1 MB`
1. Set **Minimum Time** to `0`
1. Set ** Ignore Above Size** to `500 MB`
1. Click **Save**

### Content Blocker Configuration
*I do not use this because I am also using Recyclarr (or Profilarr) to ensure my grabs are high quality. If you are not using one of those tools I would suggest enabling this feature.*

### Download Cleaner Configuration
1. Check the **Download Queue Cleaner** box
1. Set the **Run Schedule** to `1 Hour`

#### Seeding Settings
*These settings are up to you, but the below values are what I use*

1. Check the box to **Delete Private Torrents**
1. Click the button to **Add a Category**
1. Set the **Category Name** to `cleanuparr-unlinked`
1. Set the **Max Seed Time** to `360`

#### Unlinked Download Settings
*These settings are up to you, but the below values are what I use*

1. Check the box to **Enable Unlinked Download Handling**
1. Set the **Target Category** to `cleanuparr-unlinked`
1. Set the **Ignored Root Directory** to `/media/downloads`
1. Add `radarr`, `tv-sonarr` and `cross-seed-link` to **Unlinked Categories**
1. Click **Save**

## Notifications

1. Select which service you use
1. If you use **Apprise** (most likely), enter you **URL** and **API Key**

    > See the list of [Apprise Services](https://github.com/caronc/apprise/wiki)
    {.is-info}


1. Check all the categories you would like notifications for
1. Click **Save**

# Video










