---
title: OmniArr
description: A guide to deploying OmniArr
published: true
date: 2026-10-31T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-10-31T12:00:00.000Z
---

# What is OmniArr?

**OmniArr** is a media discovery and request dashboard for Jellyfin households. It brings Jellyfin, Sonarr, Radarr and TMDb together in one interface, so your users can browse the library, find something new, request it and watch it download without ever touching the \*arr apps.

- Users sign in with their own Jellyfin profile and password
- Request movies, whole series, single seasons or individual episodes, with quality profile selection
- Calendar of upcoming episodes and movies pulled from Sonarr and Radarr
- Discovery and search powered by TMDb, with optional TVDB translations
- Download queue and disk space overview from Sonarr and Radarr
- Per-user wishlists, plus backup and restore of your settings

<div class="glance">
  <div><span>Port</span><b><code>8787</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/WonderAliceUwU/OmniArr"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy OmniArr
```yaml
services:
  omniarr:
    image: wonderalice/omniarr:latest
    container_name: omniarr
    restart: unless-stopped
    ports:
      - 8787:8787
    environment:
      - PORT=8787
      - OMNIARR_DATA_DIR=/app/data
    volumes:
      - /mnt/tank/configs/omniarr:/app/data
```

1. Deploy the stack
1. Open the web UI at `http://YOUR-SERVER-IP:8787`

> OmniArr keeps everything (admin password, API keys, integration settings) in a single encrypted file inside `/app/data`. If that folder is not mounted, every setting is lost the next time the container is recreated.
{.is-warning}

> OmniArr is distributed as a private project, not under an open source license.
{.is-info}

# 2 · Configuration

## 2.1 Create the Admin Account
1. On first launch OmniArr shows a setup screen. There are no default credentials
1. Create a master password of at least 8 characters. This is the administrator login

## 2.2 Connect Your Services
1. Sign in as admin and open **Administration**
1. Fill in each integration. OmniArr tests every connection as you save it:

| Service | What you need |
|---------|---------------|
| Jellyfin | URL and an API key (**Dashboard → API Keys**) |
| Sonarr | URL, API key, root folder and quality profile |
| Radarr | URL, API key, root folder and quality profile |
| TMDb | API Read Access Token (the long bearer token, not the short API key) |
| TVDB | API key (optional, used for translations) |
{.dense}

> Get a free TMDb token by creating an account at [themoviedb.org](https://www.themoviedb.org/settings/api). Discovery and search will not work without it.
{.is-info}

## 2.3 User Logins
Once Jellyfin is connected, everyone else signs in by picking their Jellyfin profile on the login screen and entering their normal Jellyfin password. Sessions last 12 hours.
