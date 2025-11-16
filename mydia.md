---
title: Mydia
description: A guide to deploying Mydia
published: true
date: 2025-11-16T13:15:26.713Z
tags: 
editor: markdown
dateCreated: 2025-11-16T13:15:26.713Z
---

# What is Mydia?
A modern, self-hosted media management platform for tracking, organizing, and monitoring your media library.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Mydia
```yaml
services:
  mydia:
    image: ghcr.io/getmydia/mydia:latest
    container_name: mydia
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
      - SECRET_KEY_BASE=F0cBmQrlSn9DllTroJECMW3T0kYgpBvysBRzYqxlX4z3WffISuauecEk7norkU9u
      - GUARDIAN_SECRET_KEY=F0cBmQrlSn9DllTroJECMW3T0kYgpBvysBRzYqxlX4z3WffISuauecEk7norkU9u
      - PHX_HOST=10.99.0.242
      - PORT=4000
      - MOVIES_PATH=/media/movies
      - TV_PATH=/media/tv
    volumes:
      - /mnt/tank/configs/mydia:/config
      - /mnt/tank/media:/media
    ports:
      - 4000:4000
    restart: unless-stopped
```