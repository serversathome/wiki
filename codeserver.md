---
title: Code Server
description: A guide to deploying Code Server
published: true
date: '2026-11-04T16:17:00.857Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:03:50.157Z'
---

# What is Code Server?

Run VS Code on any machine anywhere and access it in the browser.

# 1 · Deploy Code Server
:::block-tabs
::block-tab{label="TrueNAS" icon="simple-icons:truenas"}
1. Set an **Additional Environment Variable** 
a. Name = PASSWORD
b. Value = *choose a password*
1. Set the **Code Server Project Storage** to *Host Path* and select your pool 

::

::block-tab{label="Docker Compose" icon="simple-icons:docker"}
```yaml
services:
  code-server:
    image: lscr.io/linuxserver/code-server:latest
    container_name: code-server
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
      - PASSWORD=admin
    volumes:
      - ./config:/config
      - /mnt/tank:/config/workspace
    ports:
      - 8443:8443
    restart: unless-stopped
```

::

:::
