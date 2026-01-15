---
title: Dockhand
description: A guide to depoying Dockhand
published: true
date: 2026-01-15T15:28:58.594Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:20.950Z
---

# <img src="/dockhand.webp" class="tab-icon"> What is Dockhand?

From simple container operations to complex multi-environment deployments.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Dockhand
```yaml
services:
  dockhand:
    image: fnsys/dockhand:latest
    container_name: dockhand
    restart: unless-stopped
    ports:
      - 3000:3000
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /mnt/tank/configs/dockhand:/app/data
      - /mnt/tank/stacks:/app/data/stacks
```
