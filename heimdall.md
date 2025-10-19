---
title: Heimdall
description: A guide to deploying Heimdall
published: true
date: 2025-10-19T23:59:32.015Z
tags: 
editor: markdown
dateCreated: 2025-10-19T23:59:32.015Z
---

# ![](/heimdall.png){class="tab-icon"} What is Heimdall?
A sleek, modern dashboard that puts all of your apps and services at your fingertips. Control everything in one convenient location. Seamlessly integrates with the apps you've added, providing you with valuable information.


# 1 · Deploy Heimdall
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  heimdall:
    image: lscr.io/linuxserver/heimdall:latest
    container_name: heimdall
    restart: unless-stopped
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
      - ALLOW_INTERNAL_REQUESTS=false #optional
    volumes:
      - /mnt/tank/configs/heimdall:/config
    ports:
      - 1080:80
      - 10443:443
```
By default, Heimdall blocks lookup requests to private or reserved IP addresses, if your instance is not exposed to the internet, or is behind some level of authentication, you can set `ALLOW_INTERNAL_REQUESTS` to true to allow requests to private IP addresses.

## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Set the **Heimdall Data Storage** to *Host Path* and select a dataset to store the config files

# <img src="/youtube.png" class="tab-icon"> 2 · Video

