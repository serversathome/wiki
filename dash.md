---
title: Dash.
description: A guide to deploying Dash.
published: true
date: '2026-11-04T16:17:04.254Z'
tags: []
editor: markdown
dateCreated: '2026-01-26T14:16:20.520Z'
---

# What is Dash.?

A simple, modern server dashboard, primarily used by smaller private servers.

# 1 · Deploy Dash.
> [!NOTE]
> Dash. reads OS, storage and network info from the host root, so `/` is mounted read-only at `/mnt/host`. It has no config folder; all settings are environment variables.

:::block-tabs
::block-tab{label="Without GPU" icon="simple-icons:docker"}
```yaml
services:
  dash:
    image: mauricenino/dashdot:latest
    restart: unless-stopped
    container_name: dash
    privileged: true
    ports:
      - '3001:3001'
    volumes:
      - /:/mnt/host:ro
    environment:
      DASHDOT_WIDGET_LIST: 'os,cpu,storage,ram,network'
```

::

::block-tab{label="With GPU" icon="simple-icons:docker"}
```yaml
services:
  dash:
    image: mauricenino/dashdot:nvidia
    restart: unless-stopped
    privileged: true
    deploy:
      resources:
        reservations:
          devices:
            - capabilities:
                - gpu
    ports:
      - '3001:3001'
    volumes:
      - /:/mnt/host:ro
    environment:
      DASHDOT_WIDGET_LIST: 'os,cpu,storage,ram,network,gpu'
```

::

:::

# 2 · Environment Variables
See list of all environment variables [here](https://getdashdot.com/docs/configuration/basic).
