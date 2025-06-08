---
title: Suggestarr
description: A guide to deploying the Suggestarr container in docker
published: true
date: 2025-06-08T18:40:32.168Z
tags: 
editor: markdown
dateCreated: 2025-05-09T10:33:58.203Z
---

# What is Suggestarr?

SuggestArr is a project designed to automate media content recommendations and download requests based on user activity in media servers like Jellyfin, Plex, and now Emby. It retrieves recently watched content, searches for similar titles using the TMDb API, and sends automated download requests to Jellyseer or Overseer.

# Installation

```yaml
services:
  suggestarr:
    image: ciuse99/suggestarr:latest
    container_name: suggestarr
    restart: unless-stopped
    ports:
      - 5000:5000
    volumes:
      - /mnt/tank/configs/suggestarr:/app/config/config_files
    environment:
      - LOG_LEVEL=info
```

# Create a Jellyseerr User

In order for Suggestarr to request media, it needs a **Local User** in Jellyseerr. 
1. Navigate to **Jellyseerr → Users → Create Local User**
1. Enter a username and a password
> 
> If you would like Suggestarr to automatically download all requested content, **Edit** the new user, click the **Permissions** tab, and check the **Auto-Approve** box
{.is-info}

# Create a TMDB API Key

This is necessary for Suggestarr to be able to browse media to recommend content.

1. Navigate to [TMDB](https://www.themoviedb.org/)
1. Create a login
1. Navigate to the user menu (circle in the top right corner)
1. Select **Settings → API**
1. Create an API Key

# Suggestarr Configuration

Navigate to your server IP and port 5000.

1. Select the media server you use
1. Enter the TMDB API Key you just created and click the ▶️ button to the right of the text box (make sure you get the ✅)
1. Enter your media server URL and API Key and click the ▶️ button to the right of the text box (make sure you get the ✅)
1. Select the media libraries you want to include
1. Select users you want to include or don't select any to use *all*
1. Enter your Jellyseerr server URL and API Key and click the ▶️ button to the right of the text box (make sure you get the ✅)
1. Select the user you created above and enter the password then click the **Authenticate** button
1. Use the default **Database Configuration**
1. Seelct the parameters of the content you would like. I suggest changing the **Region**, **Content**, **Release Year**, and probably **Number of Seasons**.
1. Set the maximum values for searches
1. Click **Run Now**

# Video

![2025-05-09-suggestarr-automatically-add-me-promo-card.png](/2025-05-09-suggestarr-automatically-add-me-promo-card.png)
[Watch it on Patreon!](https://www.patreon.com/posts/suggestarr-add-128533867?utm_medium=clipboard_copy&utm_source=copyLink&utm_campaign=postshare_creator&utm_content=join_link)
