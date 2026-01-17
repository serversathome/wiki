---
title: Docker Compose Maker
description: A guide to deploying Docker Compose Maker
published: true
date: 2026-01-17T17:34:25.366Z
tags: 
editor: markdown
dateCreated: 2026-01-17T17:33:37.687Z
---

# <img src="/dcm.png" class="tab-icon"> What is Docker Compose Maker?
DCM (Docker Compose Maker) is a simple yet powerful tool that helps you create docker-compose.yaml files for your self-hosted applications. Select from a curated list of popular containers and generate a ready-to-use configuration file with just a few clicks.

No more copy-pasting from documentation or trying to remember the correct configuration options - this tool makes it easy to set up your Docker environment.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Docker Compose Maker
```yaml
services:
  dcm:
    image: ghcr.io/ajnart/dcm
    container_name: dcm
    ports:
      - "7576:7576"
    restart: unless-stopped