---
title: Arcane
description: A guide to deploying Arcane in docker
published: true
date: 2025-11-11T19:06:18.330Z
tags: 
editor: markdown
dateCreated: 2025-06-04T16:58:11.419Z
---

# ![](/arcane.png){class="tab-icon"} What is Arcane?
Arcane is a modern easy to use way to manage your docker containers, images, volumes, and networks — all in one place.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Arcane
```yaml
services:
  arcane:
    image: ghcr.io/getarcaneapp/arcane:latest
    container_name: arcane
    restart: unless-stopped
    ports:
      - 3552:3552
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /mnt/tank/stacks:/mnt/tank/stacks
      - /mnt/tank/configs/arcane:/app/data
    environment:
      - PUID=568
      - PGID=568
      - PROJECTS_DIRECTORY=/mnt/tank/stacks
      - APP_URL=http://10.99.0.242:3552
      - ENCRYPTION_KEY=799513078c58a0163eb3cb217b1226aaad243a350043bf130b711d9770b1fc19
      - JWT_SECRET=ed012e40dbfbcad08a59b23a6456496ed353bd1b47e30d391363fa7b4e1d4adf
```
1. Pick a `configs` directory which exists already with `568` permissions

# 2 · Logging In
1. Navigate to http://{IP}:3552
1. Default user name = `arcane`
1. Default password = `arcane-admin`

# 3 · First Run Wizard
1. Set a new password
1. In the **Docker Setup** tab I recommend disabling `Auto-update Containers`
1. In the **Application Settings** tab set the `Base Server URL` to the IP of your server


# <img src="/youtube.png" class="tab-icon"> 3 · Video

[](https://youtu.be/p-sd7dAbyCo)
