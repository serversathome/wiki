---
title: AdGuard
description: A guide to deploying AdGuard via TrueNAS or docker compose
published: true
date: 2026-01-17T01:25:52.964Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:03:05.289Z
---

# ![](/adguard-home.png){class="tab-icon"} What is AdGuard?
AdGuard is the best way to get rid of annoying ads and online tracking and protect your computer from malware. Make your web surfing fast, safe and ad-free.


# 1 · Deploy AdGuard
# {.tabset}
## <img src="/truenas.png" class="tab-icon"> TrueNAS
1. Set the **Storage Configuration** to use **Host Path** for the **AdGuard Home Config Storage** and **AdGuard Home WorkDir Storage**

## <img src="/docker.png" class="tab-icon"> Docker Compose
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
      # - 68:68/udp
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

# 2 · Adguard Configuration
1. Naviagte to the IP and Port of the container
1. Leave the **Interfaces** and **DNS Server** settings default
1. Create a username and password
1. Set the DNS of your router to point to the IP of the container

## 2.1 DNS Settings
1. Add Upstream DNS Servers to the list already pre-populated

## 2.2 Encryption Settings
1. Check the box to **Enable Encryption** to use HTTPS, DNS-over-HTTPS, and DNS-over-TLS

## 2.3 DNS Blocklists
1. Optionally add additional blocklists besides the default AdGuard DNS filter list

# <img src="/youtube.png" class="tab-icon"> 3 · Video
https://youtu.be/u9PioLP57-4