---
title: Rackula
description: A guide to deploying Rackula
published: true
date: 2025-12-29T22:09:54.335Z
tags: 
editor: markdown
dateCreated: 2025-12-29T22:09:54.335Z
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