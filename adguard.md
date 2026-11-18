---
title: AdGuard
description: A guide to deploying AdGuard via TrueNAS or docker compose
published: true
date: '2026-11-18T12:12:49.548Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:03:05.289Z'
---

# What is AdGuard?
AdGuard is the best way to get rid of annoying ads and online tracking and protect your computer from malware. Make your web surfing fast, safe and ad-free.

# 1 · Deploy AdGuard
:::block-tabs
::block-tab{label="TrueNAS" icon="simple-icons:truenas"}
1. Set the **Storage Configuration** to use **Host Path** for the **AdGuard Home Config Storage** and **AdGuard Home WorkDir Storage**

::

::block-tab{label="Docker Compose" icon="simple-icons:docker"}
```yaml
services:
  adguardhome:
    image: adguard/adguardhome:latest
    container_name: adguardhome
    user: "568:568"
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/adguard/work:/opt/adguardhome/work
      - /mnt/tank/configs/adguard/conf:/opt/adguardhome/conf
    ports:
      - 53:53/tcp
      - 53:53/udp
      - 3000:3000/tcp
```

::

:::

# 2 · Adguard Configuration

::block-steps
1. Naviagte to the IP and Port of the container
1. Leave the **Interfaces** and **DNS Server** settings default
1. Create a username and password
1. Set the DNS of your router to point to the IP of the container
::

## 2.1 DNS Settings
1. Add Upstream DNS Servers to the list already pre-populated

## 2.2 Encryption Settings
1. Check the box to **Enable Encryption** to use HTTPS, DNS-over-HTTPS, and DNS-over-TLS

## 2.3 DNS Blocklists
1. Optionally add additional blocklists besides the default AdGuard DNS filter list

# :logos:youtube-icon: 3 · Video
::block-youtube{url="https://youtu.be/u9PioLP57-4"}
::
