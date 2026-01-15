---
title: Koffan
description: A guide to deploying Koffan
published: true
date: 2026-01-15T15:29:52.227Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:05:48.703Z
---

# <img src="/dockpeek.png" class="tab-icon"> What is Koffan?
Koffan is a lightweight web application for managing shopping lists, designed for couples and families. It allows real-time synchronization between multiple devices, so everyone knows what to buy and what's already in the cart.

The app works in any browser on both mobile and desktop. Just one password to log in - no complicated registration required.


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Koffan
```yaml
services:
  koffan:
    ports:
      - 3000:80
    environment:
      - APP_PASSWORD=admin
    volumes:
      - /mnt/tank/configs/koffan:/data
    image: ghcr.io/pansalut/koffan:latest
    restart: unless-stopped
    container_name: koffan
```