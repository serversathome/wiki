---
title: Sousous
description: A guide to deploying Sousous
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:08:42.424Z
---

# What is Sousous?
Sousous is a clean and lightweight personal finance app. It helps you track your expenses, categorize them, and get a quick view of your budget.

<div class="glance">
  <div><span>Port</span><b><code>4999</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://codeberg.org/tdelorge/sousous"><i class="mdi mdi-git"></i>Project</a>
  </div>
</div>

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
      FLASK_SECRET_KEY: "paste-your-generated-key-here" # Generate with `openssl rand -hex 32`
      FLASK_COOKIE_SECURE: "false"
    restart: unless-stopped
```

> Generate your own `FLASK_SECRET_KEY` with `openssl rand -hex 32` and paste it in before the first start. Don't reuse a key copied from a guide.
{.is-warning}

# 2 · Logging In
The default user is `admin` and the default password is `admin123`. Change it right after your first login.