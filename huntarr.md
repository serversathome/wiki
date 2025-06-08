---
title: Huntarr
description: A guide to deploying huntarr via docker compose
published: true
date: 2025-06-08T18:40:27.697Z
tags: 
editor: markdown
dateCreated: 2025-04-25T17:34:28.189Z
---

![huntarr.png](/huntarr.png)

# What is Huntarr?
 A specialized utility that automates discovering missing and upgrading your media collection!
 
[Github](https://github.com/plexguide/Huntarr.io)

# Installation
```yaml
services:
  huntarr:
    image: huntarr/huntarr:latest
    container_name: huntarr
    restart: unless-stopped
    ports:
      - "9705:9705"
    volumes:
      - /mnt/tank/configs/huntarr:/config
    environment:
      - TZ=America/New_York
```

# Huntarr Configuration
1. Create a login and password
> Password must be 8 characters long and include a number and an uppercase letter
{.is-info}
2. Navigate to **Apps** in the left menu and select **Sonarr** from the dropdown menu at the top
1. **Enter your URL** for Sonarr
1. Enter your **API Key** (Found in **Settings → General**)
1. Leave the **Missing Search Mode** set to **Season Packs**
1. Enter max number of **Missing Shows to Search**
1. Enter max number of **Episodes to Upgrade**
1. Leave everything else as default
1. Click **Save** at the top
1. Now do the same for Radarr

> Optionally, you can select **Swapparr** from the dropdown and click the **☑️ Enable Swaparr** checkbox leaving all settings as default to automatically remove stalled torrents
{.is-info}



# Video

![2025-04-28-huntarr--automate-missing--cut-promo-card.png](/2025-04-28-huntarr--automate-missing--cut-promo-card.png)
[See it on Patreon!](https://www.patreon.com/posts/huntarr-v7-is-130103029?utm_medium=clipboard_copy&utm_source=copyLink&utm_campaign=postshare_creator&utm_content=join_link)