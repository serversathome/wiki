---
title: Sousous
description: A guide to deploying Sousous
published: true
date: '2026-11-04T16:23:44.120Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:08:42.424Z'
---

# What is Sousous?
Sousous is a clean and lightweight personal finance app. It helps you track your expenses, categorize them, and get a quick view of your budget.

# :logos:docker-icon: 1 · Deploy Sousous
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
      FLASK_SECRET_KEY: "paste-your-generated-key-here" # Generate with `openssl rand -hex 32`
      FLASK_COOKIE_SECURE: "false"
    restart: unless-stopped
```

> [!WARNING]
> Generate your own `FLASK_SECRET_KEY` with `openssl rand -hex 32` and paste it in before the first start. Don't reuse a key copied from a guide.

# 2 · Logging In
The default user is `admin` and the default password is `admin123`. Change it right after your first login.
