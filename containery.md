---
title: Containery
description: A guide to deploying Containery in docker
published: true
date: 2025-08-04T11:17:28.215Z
tags: 
editor: markdown
dateCreated: 2025-08-04T11:17:28.215Z
---

# ![](/containery-white.png){class="tab-icon"} What is Containery?

Containery is a web-based container management tool that provides a fast, lightweight, and intuitive interface for managing Docker containers. Whether you're a software engineer, DevOps, QA, or anyone who needs to interact with containers, Containery makes it easy to monitor status, view logs, and access terminals for quick insights and control.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Containery

```yaml
services:
  app:
    image: ghcr.io/danylo829/containery:latest
    container_name: containery
    restart: "unless-stopped"
    ports:
      - "5000:5000"
    volumes:
      - /mnt/tank/configs/containery:/containery_data
      - /mnt/tank/configs/containery/containery_static:/containery/app/static/dist
      - /var/run/docker.sock:/var/run/docker.sock:ro
```
