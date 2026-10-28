---
title: Cleanuparr
description: A guide to deploying Cleanuparr via docker
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:03:41.816Z
---

# ![](/cleanuparr.png){class="tab-icon"} What is Cleanuparr?

Automated Download Management. Automatically clean up unwanted, stalled, and malicious downloads from your \*arr applications and download clients. Keep your queues clean and your media library safe.

<div class="glance">
  <div><span>Port</span><b><code>11011</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/cleanuparr/cleanuparr"><i class="mdi mdi-github"></i>Project</a>
    <a href="https://cleanuparr.github.io/docs/"><i class="mdi mdi-book-open-variant"></i>Docs</a>
  </div>
</div>


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Cleanuparr

> Read the [official documentation](https://cleanuparr.github.io/docs/)
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
- **Volumes**: The container structure follows a common-sense naming convention, storing configurations under /mnt/tank/configs/cleanuparr
- 📌 Refer to the [Folder-Structure](/Folder-Structure) guide for more details.


# 2 · Cleanuparr Configuration
> If at any time you do not know what a setting does, click the ⓘ symbol next to it to be taken to the official document page explaining it
{.is-info}

## 2.1 Instances
1. Open the **Media Managers** page on the left pane. All your \*arr apps are managed from this one page
1. Add each instance, choosing its type, then enter the instance name, URL and API key

## 2.2 Download Clients
1. Add your download client by navigating to the **Download Clients** tab on the left pane. qBittorrent, SABnzbd and others are supported
1. Enter your instance name, client type, host (URL), and credentials (qBittorrent can also use an API key instead of username and password)

> The client type can't be changed after the client is created. To switch types, delete it and add a new one.
{.is-info}

## 2.3 Cleanup
### 2.3.1 General Configuration
1. Uncheck the box for **Display Support Banner** for a cleaner dashboard
1. Change the **Certificate Validation** to **Disabled for Local Addresses** 
1. Click **Save**

### 2.3.2 Queue Cleaner Configuration
1. Check the **Enable Queue Cleaner** box

#### 2.3.2.1 Failed Import Settings
*These settings are up to you, but the below values are what I use*

1. Set **Max Strikes** to `3`
1. Check the box for **Ignore Private**

#### 2.3.2.2 Stalled Download Settings
*These settings are up to you, but the below values are what I use*

1. Set **Max Strikes** to `3`
1. Check the box for **Reset Strikes On Progress**
1. Check the box for **Ignore Private**

#### 2.3.3.3 Slow Download Settings
*These settings are up to you, but the below values are what I use*

1. Set **Max Strikes** to `3`
1. Check the box for **Ignore Private**
1. Set **Minimum Speed** to `1 MB`
1. Set **Minimum Time** to `0`
1. Set ** Ignore Above Size** to `500 MB`
1. Click **Save**

### 2.3.3 Content Blocker Configuration
*I do not use this because I am also using [Profilarr](/profilarr) to ensure my grabs are high quality. If you are not using it I would suggest enabling this feature.*

### 2.3.4 Download Cleaner Configuration
1. Check the **Download Queue Cleaner** box
1. Set the **Run Schedule** to `1 Hour`

#### 2.3.4.1 Seeding Settings
*These settings are up to you, but the below values are what I use*

1. Check the box to **Delete Private Torrents**
1. Click the button to **Add a Category**
1. Set the **Category Name** to `cleanuparr-unlinked`
1. Set the **Max Seed Time** to `360`

#### 2.3.4.2 Unlinked Download Settings
*These settings are up to you, but the below values are what I use*

1. Check the box to **Enable Unlinked Download Handling**
1. Set the **Target Category** to `cleanuparr-unlinked`
1. Set the **Ignored Root Directory** to `/media/downloads`
1. Add `radarr`, `tv-sonarr` and `cross-seed-link` to **Unlinked Categories**
1. Click **Save**

## 2.4 Notifications

1. Select which service you use
1. If you use **Apprise** (most likely), enter you **URL** and **API Key**
1. Check all the categories you would like notifications for
1. Click **Save**

> See the list of [Apprise Services](https://github.com/caronc/apprise/wiki) to get your URL
{.is-info}


# <img src="/patreon-light.png" class="tab-icon"> 3 · Video

[![2025-06-30-how-to-install-and-configure-cle-promo-card.png](/2025-06-30-how-to-install-and-configure-cle-promo-card.png)](https://www.patreon.com/posts/how-to-install-132884071)






