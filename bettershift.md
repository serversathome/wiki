---
title: Bettershift
description: A guide to deploying Bettershift
published: true
date: 2025-12-29T22:23:50.035Z
tags: 
editor: markdown
dateCreated: 2025-12-29T22:23:50.035Z
---

# What is Bettershift?
BetterShift is a modern shift management application designed to simplify variable work schedules. Manage unlimited calendars with one-click shift toggles, reusable presets, and real-time synchronization across devices. Features include external calendar integration (Google, Outlook, iCal), password-protected calendars, ICS/PDF export, live statistics, calendar comparison and multi-language support.


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Bettershift
```yaml
services:
  bettershift:
    ports:
      - 3000:3000
    volumes:
      - /mnt/tank/configs/bettershift:/app/data
    container_name: bettershift
    image: ghcr.io/pantelx/bettershift:latest
```