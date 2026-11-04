---
title: CrossWatch
description: A guide to deploying CrossWatch
published: true
date: '2026-11-04T16:24:06.752Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:04:10.399Z'
---

# What is CrossWatch?
Synchronize your data across Plex, Jellyfin, SIMKL, Trakt, and more. Keep your movies and shows in sync, no matter where you watch. 

# :logos:docker-icon: 1 · Deploy CrossWatch

```yaml
services:
  crosswatch:
    image: ghcr.io/cenodude/crosswatch:latest
    container_name: crosswatch
    ports:
      - 8787:8787
    environment:
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/crosswatch:/config
    restart: unless-stopped
```

