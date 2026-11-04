---
title: Rackula
description: A guide to deploying Rackula
published: true
date: '2026-11-04T16:23:28.830Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:07:51.532Z'
---

# What is Rackula?
Plan your rack layout. Drag your devices in, move them around, export it. It runs in your browser. You can close the tab whenever you want.

# :logos:docker-icon: 1 · Deploy Rackula
```yaml
services:
  rackula:
    image: ghcr.io/rackulalives/rackula:latest
    container_name: rackula
    ports:
      - "8080:8080"
    restart: unless-stopped
```

> [!NOTE]
> By default your layouts live in the browser. If you want them saved on the server (with optional local or OIDC login), use the project's [persistence compose](https://github.com/RackulaLives/Rackula/blob/main/docs/deployment/SELF-HOSTING.md), which adds the `rackula-api` sidecar.
