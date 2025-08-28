---
title: Sousous
description: A guide to deploying Sousous
published: true
date: 2025-08-28T19:06:49.347Z
tags: 
editor: markdown
dateCreated: 2025-08-28T19:06:49.347Z
---

# What is Sousous?
Sousous is a clean and lightweight personal finance app. It helps you track your expenses, categorize them, and get a quick view of your budget.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Sousous
```yaml
services:
  sousous:
    image: codeberg.org/tdelorge/sousous:latest
    container_name: sousous
    ports:
      - "4999:5000"
    volumes:
      - /mnt/tank/configs/sousous/:/app/src/db
    environment:
      PUID: 568
      PGID: 568
      TZ: "America/New_York"
      FLASK_SECRET_KEY: "plxzTM5B8lIxBAkTUhQnkc131y5r2UmR7U8LsfnU"
      FLASK_COOKIE_SECURE: "false"
    restart: unless-stopped
```

# 2 · Logging In
The default user is `admin` and the default password is `admin123`.