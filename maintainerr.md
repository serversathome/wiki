---
title: Maintainerr
description: A guide to deploying Maintainer via docker
published: true
date: 2025-07-14T12:07:00.413Z
tags: 
editor: markdown
dateCreated: 2025-07-14T11:50:00.697Z
---

# ![Radarr](/maintainerr.png){class="tab-icon"} What is Maintainerr?

Maintainerr makes managing your media easy.

- Do you hate being the janitor of your server?
- Do you have a lot of media that never gets watched?
- Do your users constantly request media, and let it sit there afterward never to be touched again?

If you answered yes to any of those questions.. You NEED Maintainerr. It's a one-stop-shop for handling those outlying shows and movies that take up precious space on your server.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Radarr
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