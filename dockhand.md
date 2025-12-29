---
title: Dockhand
description: A guide to depoying Dockhand
published: true
date: 2025-12-29T19:19:38.460Z
tags: 
editor: markdown
dateCreated: 2025-12-29T19:19:38.460Z
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
      - /mnt/tank.configs.dockhand:/app/data
      - /mnt/tank/stacks:/app/data/stacks
```