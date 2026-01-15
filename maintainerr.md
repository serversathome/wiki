---
title: Maintainerr
description: A guide to deploying Maintainer via docker
published: true
date: 2026-01-15T15:30:04.001Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:06:10.333Z
---

# ![Radarr](/maintainerr.png){class="tab-icon"} What is Maintainerr?

Maintainerr makes managing your media easy.

- Do you hate being the janitor of your server?
- Do you have a lot of media that never gets watched?
- Do your users constantly request media, and let it sit there afterward never to be touched again?

If you answered yes to any of those questions.. You NEED Maintainerr. It's a one-stop-shop for handling those outlying shows and movies that take up precious space on your server.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Maintainerr
```yaml
services:
    maintainerr:
        image: ghcr.io/jorenn92/maintainerr:latest 
        container_name: maintainerr
        user: 568:568
        volumes:
          - /mnt/tank/configs/maintainerr:/opt/data
        environment:
          - TZ=America/New_York
        ports:
          - 6246:6246
        restart: unless-stopped
```

# 2 · Maintainerr Configuration

> Read the [official documentation](https://docs.maintainerr.info/latest/#__consent)!
{.is-success}

1. At a minimum, add Plex, Radarr, and Sonarr servers
2. Create rules to define how Maintainerr will handle media

# <img src="/youtube.png" class="tab-icon"> 3 · Video
https://youtu.be/u8k-IlkShKs