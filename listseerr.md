---
title: Listseerr
description: A guide to deploying Listseerr
published: true
date: '2026-11-04T16:24:22.597Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:06:04.583Z'
---

# What is Listseerr?

Request movies & shows in Seerr from your favorite lists.

# :logos:docker-icon: 1 · Deploy Listseerr
```yaml
services:
  listseerr:
    image: ghcr.io/guillevc/listseerr:latest
    container_name: listseerr
    ports:
      - 3000:3000
    environment:
      TZ: 'America/New_York'
      # (REQUIRED) Generate with: openssl rand -hex 32
      ENCRYPTION_KEY: 'CHANGE_ME'
    volumes:
      - /mnt/tank/configs/listseerr:/app/data
    restart: unless-stopped
```

# ![](/_files/patreon-light.png =28x) 2 · Video

[![](/2026-01-13-add-all-the-lists-with-listseerr-promo-card.png)](https://www.patreon.com/posts/add-all-lists-147714038)
