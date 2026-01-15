---
title: Auto-Limit
description: A guide to deploying Auto-Limit via docker
published: true
date: 2026-01-15T15:28:08.017Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:03:16.164Z
---

# What is Auto-Limit?
Auto-Limit is an intelligent download speed management tool designed specifically for NAS users and home media server enthusiasts. It automatically reduces the speed of downloaders like qBittorrent and Transmission when family members are watching movies on media servers like Emby or Jellyfin, ensuring smooth streaming without buffering.

# Installation
```yaml
services:
  auto-limit:
    image: xiaobaiya000/auto-limit:latest
    container_name: auto-limit
    ports:
      - 9190:9190
    volumes:
      - /mnt/tank/configs/autolimit:/app/data
    environment:
      - TZ=America/New_York
    restart: unless-stopped
```

# Auto-Limit Configuration
1. Navigate to the IP and Port of the container
1. Create a username and password
1. Click the **Configuration** tab
1. Click **Add Media Server**
	a. Give it an **Instance Name**
  b. Select the **Plugin Type**
  c. Add your **API Key**
  d. Click **Save**
1. Click **Add Downloader**
	a. Give it an **Instance Name**
  b. Select the **Plugin Type**
  c. Add the **Server Address**
  d. Enter your **Username** and **Password**
  e. Set the **Download/Upload During Playback** speeds to the `KB/s` your network can handle
  
# Video

[![](/2025-07-14-fix-buffering-forever-auto-limi-promo-card.png)](https://www.patreon.com/posts/fix-buffering-133668383)