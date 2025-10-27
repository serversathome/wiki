---
title: Dashy
description: A guide to deploying Dashy
published: true
date: 2025-10-27T20:57:44.719Z
tags: 
editor: markdown
dateCreated: 2025-10-27T20:57:44.719Z
---

# ![](/dashy.png){class="tab-icon"} What is Dashy?
A self-hostable personal dashboard built for you. Includes status-checking, widgets, themes, icon packs, a UI editor and tons more! 

# 1 · Deploy Dashy
# {.tabset}
## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Set the **Dashy Config Storage** to **Host Path**

## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  dashy:
    container_name: Dashy
    image: lissy93/dashy
    volumes:
      - /mnt/tank/configs/dashy/my-config.yml:/app/user-data/conf.yml
      # - /path/to/item-icons:/app/user-data/item-icons/
    ports:
      - 4000:8080
    environment:
      - NODE_ENV=production
      - UID=568
      - GID=568
    restart: unless-stopped
```