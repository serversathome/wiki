---
title: Dashy
description: A guide to deploying Dashy
published: true
date: '2026-11-04T16:17:04.936Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:04:14.446Z'
---

# What is Dashy?
A self-hostable personal dashboard built for you. Includes status-checking, widgets, themes, icon packs, a UI editor and tons more! 

# 1 · Deploy Dashy
:::block-tabs
::block-tab{label="TrueNAS" icon="simple-icons:truenas"}
1. Set the **Dashy Config Storage** to **Host Path**

::

::block-tab{label="Docker Compose" icon="simple-icons:docker"}
### Docker Compose
```yaml
services:
  dashy:
    container_name: Dashy
    image: lissy93/dashy
    user: "568:568"
    volumes:
      - /mnt/tank/configs/dashy:/app/user-data/
    ports:
      - 4000:8080
    environment:
      - NODE_ENV=production
    restart: unless-stopped
```

### Sample Config File
> [!WARNING]
> Save the sample below as `/mnt/tank/configs/dashy/conf.yml` **before** starting the container. Dashy needs a `conf.yml` in the mounted folder, and an empty folder hides the default config.

```yaml

appConfig:
  theme: colorful
  layout: auto
  title: "My Dashboard"
  iconSize: medium
  headerStyle: boxed
  hideVersion: true

pageInfo:
  title: "My Dashboard"
  description: "Centralized access to all my self-hosted services"
  navLinks: []

sections:
  - name: "Section 1"
    icon: "fa-solid fa-folder"
    items: []
```

::

:::
