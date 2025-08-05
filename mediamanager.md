---
title: Media Manager
description: A guide to deploying Media Manager via docker
published: true
date: 2025-08-05T19:34:30.897Z
tags: 
editor: markdown
dateCreated: 2025-08-04T11:59:15.707Z
---

# ![](/mediamanager.png){class="tab-icon"} What is Media Manager?
MediaManager is modern software to manage your TV and movie library. It is designed to be a replacement for Sonarr, Radarr, Overseer, and Jellyseer. It supports TVDB and TMDB for metadata, supports OIDC and OAuth 2.0 for authentication and supports Prowlarr and Jackett. 

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Media Manager
```yaml
services:
  mediamanager:
    image: ghcr.io/maxdorninger/mediamanager/mediamanager:latest
    container_name: mediamanager
    restart: unless-stopped
    ports:
      - "8000:8000"
    environment:
      - CONFIG_DIR=/app/config
    volumes:
      - /mnt/tank/media:/data/
      - /mnt/tank/configs/mediamanager:/app/config/
      - /mnt/tank/configs/mediamanager/images/:/data/images/
      
  db:
    image: postgres:latest
    restart: unless-stopped
    container_name: mediamanager-db
    volumes:
      - /mnt/tank/configs/mediamanager/postgres:/var/lib/postgresql/data
    healthcheck:
      test: [ "CMD-SHELL", "pg_isready -d $${POSTGRES_DB} -U $${POSTGRES_USER}" ]
      interval: 10s
      timeout: 5s
      retries: 5
    environment:
      POSTGRES_USER: MediaManager
      POSTGRES_DB: MediaManager
      POSTGRES_PASSWORD: MediaManager

```

> Read the [official documentation!](https://maxdorninger.github.io/MediaManager/introduction.html)
{.is-success}

# 2 · Media Manager Configuration

1. Edit the `config.toml' file by entering this command in the TrueNAS shell:
    ```bash
    nano /mnt/tank/configs/mediamanager/config.toml
    ```
1. Edit the `frontend_url` and `cors_urls` lines by removing `localhost` and adding the IP of your server
1. Add the qBittorrent URL, username and password and set to `true`
1. Add the Prowlarr URL and API key and set to `true`
1. Restart the docker container
1. Navigate to the IP and port of the container and login with `admin@example.com` and password `admin`

# <img src="/patreon-light.png" class="tab-icon"> 3 · Video