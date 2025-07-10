---
title: Arcane
description: A guide to deploying Arcane in docker
published: true
date: 2025-07-10T19:56:28.345Z
tags: 
editor: markdown
dateCreated: 2025-06-04T16:58:11.419Z
---

> This container is in the process of being completely rewritten by the developer
{.is-info}


# ![](/arcane.png){class="tab-icon"} What is Arcane?
Arcane is a Simple and Elegant Docker Management UI written in Typescript and SvelteKit. It aims to provide an intuitive interface for managing your Docker containers, images, volumes, and networks.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Arcane
```yaml
services:
  arcane:
    image: ghcr.io/ofkm/arcane:latest
    container_name: arcane
    ports:
      - '3000:3000'
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /mnt/tank/configs/arcane:/app/data
      - /mnt/tank/stacks:/app/data/stacks
    environment:
      - APP_ENV=production
      - PUID=568
      - PGID=568
      - PUBLIC_SESSION_SECRET=8s2xLw4fzOInjsBWCTCTSJuGLGWNN3kyzuk0dXm5354=
    restart: unless-stopped

```
1. Pick a `configs` directory which exists already with `568` permissions
1. Choose your existing `stacks` directory

> I have noticed if you run this in conjunction with Dockge and you use the `./` to save your data locally in the `stacks` directory, Arcane will **overwrite** the permissions to `apps:apps` and break your install! *Hostpath is not affected by this.*
{.is-danger}

# 2 · Logging In
1. Navigate to http://{IP}:3000
1. Default user name = `arcane`
1. Default password = `arcane-admin`

# <img src="/youtube.png" class="tab-icon"> 3 · Video

[](https://youtu.be/p-sd7dAbyCo)
