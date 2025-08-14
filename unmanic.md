---
title: Unmanic
description: A guide to deploying Unmanic
published: true
date: 2025-08-14T19:13:20.385Z
tags: 
editor: markdown
dateCreated: 2025-08-14T19:02:47.616Z
---

# ![](/unmanic.png){class="tab-icon"} What is Unmanic?

Tdarr is a tool that can help you optimize your media files by transcode, remux, remove unwanted streams and more. It supports cross-platform nodes, hardware transcoding, plugins and job reports.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Unmanic

```yaml
services:
  unmanic:
    container_name: unmanic
    restart: unless-stopped
    image: josh5/unmanic:latest
    runtime: nvidia
    ports:
      - 8870:8888
    environment:
      - PUID=568
      - PGID=568
      - NVIDIA_VISIBLE_DEVICES=all
    volumes:
      - /mnt/tank/configs/unmanic:/config
      - /mnt/tank/media/:/library
      - ./temp:/tmp/unmanic
```

# 2 · Unmanic Configuration
1. Upon first login, expand the left pane with the hamburger menu ☰ in the top left
1. Click on **Settings**
1. Click on **Library**
	a. Under **Libraries** click on the **Settings** button
	b. Set the **Library Path** to your movies directory
	c. Turn on the switch for **Enable library scanner for this library**
	d. Turn on the switch for **Enable file monitor for this library**
1. Click the + Icon under **Libraries** to add a new library
	a. Follow the same steps above except this time set your **Library Path** to your TV directory
1. 
