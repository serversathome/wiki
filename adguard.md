---
title: Adguard
description: A guide to deploying Adguard via TrueNAS or docker compose
published: true
date: 2025-07-07T17:31:03.932Z
tags: 
editor: markdown
dateCreated: 2025-07-07T17:31:03.932Z
---

![adguard-home.png](/adguard-home.png)

# What is Adguard?
AdGuard is the best way to get rid of annoying ads and online tracking and protect your computer from malware. Make your web surfing fast, safe and ad-free.


# Installation
# {.tabset}
## TrueNAS
1. Set the **Storage Configuration** to use **Host Path** for the **AdGuard Home Config Storage** and **AdGuard Home WorkDir Storage**

## Docker Compose
```yaml
services:
  adguardhome:
    container_name: adguardhome
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/adguard/work:/opt/adguardhome/work
      - /mnt/tank/configs/adguard/conf:/opt/adguardhome/conf
    ports:
      - 53:53/tcp
      - 53:53/udp
      - 67:67/udp
      - 68:68/udp
      - 80:80/tcp
      - 443:443/tcp
      - 443:443/udp
      - 3000:3000/tcp
      - 853:853/tcp
      - 853:853/udp
      - 5443:5443/tcp
      - 5443:5443/udp
      - 6060:6060/tcp
    image: adguard/adguardhome
```
