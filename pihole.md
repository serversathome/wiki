---
title: Pi-Hole
description: A guide to deploying Pi-Hole
published: true
date: 2025-08-11T12:00:04.102Z
tags: 
editor: markdown
dateCreated: 2025-08-11T12:00:04.102Z
---

# ![](/pi-hole.png){class="tab-icon"} What is Pi-Hole?
Pi-hole is a software that blocks ads and trackers across your entire network.

# 1 · Deploy Pi-Hole
# {.tabset}
## <img src="/truenas.png" class="tab-icon"> TrueNAS
1. Set the **Storage Configuration** to use **Host Path** for the **Pi-Hole Home Config Storage** and **Pi-Hole Home WorkDir Storage**

## <img src="/docker.png" class="tab-icon"> Docker Compose
```yaml
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80/tcp"
      - "443:443/tcp"
    environment:
      TZ: 'America/New_York'
      FTLCONF_webserver_api_password: 'changeme'
      FTLCONF_dns_listeningMode: 'all'
    volumes:
      - '/mnt/tank/configs/pihole:/etc/pihole'
    restart: unless-stopped
```

# 2 · P-Hole Configuration


# <img src="/youtube.png" class="tab-icon"> 3 · Video




