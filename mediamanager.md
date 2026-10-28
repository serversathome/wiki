---
title: Media Manager
description: A guide to deploying Media Manager via docker
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:06:23.142Z
---

# ![](/mediamanager.png){class="tab-icon"} What is Media Manager?
MediaManager is modern software to manage your TV and movie library. It is designed to be a replacement for Sonarr, Radarr, Overseer, and Jellyseer. It supports TVDB and TMDB for metadata, supports OIDC and OAuth 2.0 for authentication and supports Prowlarr and Jackett. 

<div class="glance">
  <div><span>Port</span><b><code>8000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>2 services</b></div>
  <div><span>Depends on</span><b>Postgres</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/maxdorninger/mediamanager"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Media Manager
```yaml
services:
  mediamanager:
    image: quay.io/maxdorninger/mediamanager:latest
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
    depends_on:
      db:
        condition: service_healthy

  db:
    container_name: mediamanager_postgres
    image: postgres:17
    restart: unless-stopped
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

> The image is published on both Quay.io (`quay.io/maxdorninger/mediamanager`, now the primary registry) and GHCR (`ghcr.io/maxdorninger/mediamanager/mediamanager`). They are built from the same source with the same tags, so either works.
{.is-info}

> Read the [official documentation!](https://maxdorninger.github.io/MediaManager/latest/)
{.is-success}

# 2 · Media Manager Configuration

1. Edit the `config.toml' file by entering this command in the TrueNAS shell (in the event this file is empty [use this example](https://raw.githubusercontent.com/maxdorninger/MediaManager/refs/heads/master/config.example.toml)):
    ```bash
    nano /mnt/tank/configs/mediamanager/config.toml
    ```
1. Edit the `frontend_url` and `cors_urls` lines by removing `localhost` and adding the IP of your server
1. In the `[auth]` section, replace `admin@example.com` in `admin_emails` with your own email **before the first start**. The first email in this list becomes the default admin account
1. In the `[auth]` section, set `token_secret` to a random string. Generate one with:
    ```bash
    openssl rand -hex 32
    ```
1. Add the qBittorrent URL, username and password and set to `true`
1. Add the Prowlarr URL and API key and set to `true`
1. Restart the docker container

# 3 · Logging In
1. On first start MediaManager creates a default admin user and prints its credentials in the container logs:
    ```bash
    docker logs mediamanager
    ```
1. Navigate to `http://IP:8000` and log in with those credentials
1. Change the admin password after your first login

> Newer releases ship with sign-up disabled (`registration_enabled = false` in `config.toml`). To let other people register accounts, set `registration_enabled = true` in the `[auth]` section and restart the container.
{.is-info}

# <img src="/patreon-light.png" class="tab-icon"> 4 · Video

[![](/2025-08-05-mediamanager-one-app-to-replace-promo-card.png)](https://www.patreon.com/posts/mediamanager-one-135812951)