---
title: Listseerr
description: A guide to deploying Listseerr
published: true
date: 2026-01-07T20:19:47.374Z
tags: 
editor: markdown
dateCreated: 2026-01-07T20:18:16.861Z
---

# <img src="/listseerr-light.png" class="tab-icon"> What is Listseerr?

Request movies & shows in Seerr from your favorite lists.


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Listseerr
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
      ENCRYPTION_KEY: 'e31a4c1b30373bc6eb08826541c10ff650921d8f96ddf69809328e64dda56a92'
    volumes:
      - /mnt/tank/configs/listseerr:/app/data
    restart: unless-stopped
```

# <img src="/patreon-light.png" class="tab-icon"> 2 · Video