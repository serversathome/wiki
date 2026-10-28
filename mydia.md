---
title: Mydia
description: A guide to deploying Mydia
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:06:29.542Z
---

# What is Mydia?
A modern, self-hosted media management platform for tracking, organizing, and monitoring your media library.

<div class="glance">
  <div><span>Port</span><b><code>4000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/getmydia/mydia"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Mydia
```yaml
services:
  mydia:
    image: ghcr.io/getmydia/mydia:latest
    container_name: mydia
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
      - SECRET_KEY_BASE=your-secret-key-base-here # generate with: openssl rand -base64 48
      - GUARDIAN_SECRET_KEY=your-guardian-secret-key-here # generate a second, different value with: openssl rand -base64 48
      - PHX_HOST=10.99.0.242
      - PORT=4000
      - MOVIES_PATH=/media/movies
      - TV_PATH=/media/tv
    volumes:
      - /mnt/tank/configs/mydia:/config
      - /mnt/tank/media:/media
    ports:
      - 4000:4000
    restart: unless-stopped
```

> Generate two separate secrets (one for `SECRET_KEY_BASE`, one for `GUARDIAN_SECRET_KEY`) by running `openssl rand -base64 48` twice in a shell. Don't reuse the same value for both.
{.is-warning}

1. Navigate to `http://IP:4000`
1. Create your admin account

# <img src="/patreon-light.png" class="tab-icon"> 2 · Video
[![](/2025-11-24-mydia-a-modern-self-hosted-medi-promo-card.png)](https://www.patreon.com/posts/mydia-modern-143868943)