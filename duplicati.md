---
title: Duplicati
description: A guide to deploying Duplicati on TrueNAS as well as via docker compose
published: true
date: 2025-07-28T10:29:06.435Z
tags: 
editor: markdown
dateCreated: 2025-07-28T10:28:55.007Z
---

# ![](/.png){class="tab-icon"} What is Duplicati?

The seamless way for your Plex and Emby users to request new content. Ombi integrates with your media server and automatically manages user requests.

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