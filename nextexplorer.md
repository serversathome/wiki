---
title: nextExplorer
description: A guide to deploying nextExplorer
published: true
date: '2026-11-04T16:23:12.824Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:06:42.032Z'
---

# What is nextExplorer?

A modern, self-hosted file explorer with secure access control, polished UX, and a Docker-friendly deployment story.

# :logos:docker-icon: 1 · Deploy nextExplorer
```yaml
services:
  nextexplorer:
    image: nxzai/explorer:latest
    container_name: nextexplorer
    restart: unless-stopped
    ports:
      - 3000:3000
    environment:
      - NODE_ENV=production
      - PUID=568
      - PGID=568
      - PUBLIC_URL=http://<server-ip>:3000 # or your FQDN, no trailing slash
    volumes:
      - /mnt/tank/configs/nextexplorer/config:/config
      - /mnt/tank/configs/nextexplorer/cache:/cache
      # Each /mnt/<Label> mount becomes a top-level volume in the UI
      - /mnt/tank:/mnt/tank
```

> [!WARNING]
> Since v4.0, CORS and origin handling are strict. `PUBLIC_URL` must match the URL you open in the browser, or logins and uploads can fail. If you also reach it from a second address (for example a LAN IP behind a reverse proxy), add it with `INTERNAL_URL` (comma-separated). See the [release notes](https://github.com/nxzai/nextExplorer/releases).
