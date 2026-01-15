---
title: Rackula
description: A guide to deploying Rackula
published: true
date: 2026-01-15T15:31:14.756Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:51.532Z
---

# <img src="/rackula.png" class="tab-icon"> What is Rackula?
Plan your rack layout. Drag your devices in, move them around, export it. It runs in your browser. You can close the tab whenever you want.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Rackula
```yaml
services:
  Rackula:
    image: ghcr.io/rackulalives/rackula:latest
    ports:
      - "8080:80"
    restart: unless-stopped
```