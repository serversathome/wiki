---
title: Suggestarr
description: A guide to deploying the Suggestarr container in docker
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:08:48.830Z
---

# What is Suggestarr?

SuggestArr is a project designed to automate media content recommendations and download requests based on user activity in media servers like Jellyfin, Plex, and now Emby. It retrieves recently watched content, searches for similar titles using the TMDb API, and sends automated download requests to [Seerr](/seerr) (the successor to Jellyseerr and Overseerr). Newer versions can also use AI recommendations through any OpenAI-compatible LLM, pull watch history from Trakt or Simkl, and pause jobs while Seerr requests are pending.

<div class="glance">
  <div><span>Port</span><b><code>5000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/giuseppe99barchetta/SuggestArr"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Suggestarr

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
      - SUGGESTARR_PORT=5000
      - TZ=America/New_York
```

# 2 · Create a Seerr User

In order for Suggestarr to request media, it needs a **Local User** in Seerr. 
1. Navigate to **Seerr → Users → Create Local User**
1. Enter a username and a password
> 
> If you would like Suggestarr to automatically download all requested content, **Edit** the new user, click the **Permissions** tab, and check the **Auto-Approve** box
{.is-info}

# 3 · Create a TMDB API Key

This is necessary for Suggestarr to be able to browse media to recommend content.

1. Navigate to [TMDB](https://www.themoviedb.org/)
1. Create a login
1. Navigate to the user menu (circle in the top right corner)
1. Select **Settings → API**
1. Create an API Key

# 4 · Suggestarr Configuration

Navigate to your server IP and port 5000.

1. Create the first **admin account**
1. Follow the setup wizard and enter the TMDB API Key you just created
1. Select the media server you use (Plex, Jellyfin, or Emby) and enter its URL and API Key/token
1. Enter your Seerr server URL and API Key
1. Select the users and media libraries you want to include
1. Save the configuration
1. Have each user link their media server account from their **Profile**
1. Go to the **Jobs** page and create a job. Select the parameters of the content you would like. I suggest changing the **Region**, **Content**, **Release Year**, and probably **Number of Seasons**, and set a schedule
1. Run the job as a dry run first and check the results in Seerr before turning on full automation

> Use your server's IP (not `localhost`) for the media server and Seerr URLs. Inside the container, `localhost` points at SuggestArr itself.
{.is-info}

# 5 · Video

[![](/2025-05-09-suggestarr-automatically-add-me-promo-card.png)](https://www.patreon.com/posts/suggestarr-add-128533867)