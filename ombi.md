---
title: Ombi
description: A guide to deploying Ombi via docker compose
published: true
date: 2025-07-23T18:35:49.968Z
tags: 
editor: markdown
dateCreated: 2025-07-23T14:27:14.815Z
---

# ![Ombi](/ombi.png){class="tab-icon"} What is Ombi?

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

### Permissions & Folder Structure {.is-success}

* **PUID / PGID** – media‑owner UID/GID (TrueNAS SCALE default **568:568**).
* **Volumes** – configs at `/mnt/tank/configs/ombi`
  📌 See the [Folder‑Structure](/Folder-Structure) guide.

# 2 · First‑Run Configuration



# <img src="/patreon-light.png" class="tab-icon"> 2 · Video

[![](/2025-07-23-self-host-ombi-the-media-reques-promo-card.png)](https://www.patreon.com/posts/self-host-ombi-134798112)