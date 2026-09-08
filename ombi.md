---
title: Ombi
description: A guide to deploying Ombi via docker compose
published: true
date: 2026-01-15T15:30:33.053Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:06:55.304Z
---

# ![Ombi](/ombi.png){class="tab-icon"} What is Ombi?

The seamless way for your Plex and Emby users to request new content. Ombi integrates with your media server and automatically manages user requests.

<div class="glance">
  <div><span>Port</span><b><code>3579</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/linuxserver/docker-ombi"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Ombi
```yaml
services:
  ombi:
    image: lscr.io/linuxserver/ombi:latest
    container_name: ombi
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/ombi:/config
    ports:
      - 3579:3579
    restart: unless-stopped
```

### Permissions & Folder Structure

* **PUID / PGID** – media‑owner UID/GID (TrueNAS SCALE default **568:568**).
* **Volumes** – configs at `/mnt/tank/configs/ombi`
  📌 See the [Folder‑Structure](/Folder-Structure) guide.

# 2 · First‑Run Configuration

1. Select SQLite DB
1. Choose which media server you would like Ombi to manage and enter credentials
1. Create an Ombi username and password
1. Click **Next** to leave all fields blank for **Customize your Ombi** (optional)
1. Click **Finish**

# 3 · Ombi Configuration

1. Navigate to **Settings** in the left pane
1. Navigate to the **Media Server** tab
1. Click the **Load Libraries** button
1. Click **Submit**
1. Navigate to the **Configuration → User Management** tab
1. Toggle the **Import Users** to `on`
1. Click **Run Importer**
1. Click **Submit**

## 3.1 Sync \*arr Apps
1. Navigate to **TV** or **Movies** tab
1. Switch **Enable** to `on`
1. Enter the IP or hostname of the container
1. Enter the port
1. Enter the API key
1. Click **Test Connectivity**
1. Click **Load Qualities**, **Load Folders**, and **Load Tags**
1. Select the desired default quality profile
1. Select the root folder
1. Click **Submit**

# 4 · Optional Configuration

1. Navigate to the **Configuration** tab and deselect `Allow anonymous usage collection`
1. Navigate to the **Notifications** tab and configure a notification method

# <img src="/patreon-light.png" class="tab-icon"> 5 · Video

[![](/2025-07-23-self-host-ombi-the-media-reques-promo-card.png)](https://www.patreon.com/posts/self-host-ombi-134798112)