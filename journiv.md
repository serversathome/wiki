---
title: Journiv
description: A guide to deploy Journiv
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:05:43.311Z
---

# What is Journiv?
Journiv is a self-hosted private journal. It features comprehensive journaling capabilities including mood tracking, prompt-based journaling, media uploads, analytics, and advanced search with a clean and minimal UI.

<div class="glance">
  <div><span>Port</span><b><code>8000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/journiv/journiv-app"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Journiv
```yaml
services:
  journiv-app:
    container_name: journiv
    ports:
      - 8000:8000
    environment:
      - SECRET_KEY=your-secret-key-here # generate with: openssl rand -hex 32
      - DOMAIN_NAME=10.99.0.242
      - ALLOW_INSECURE_COOKIE_AUTH_OVER_HTTP=true # LAN-only, see the warning below
    volumes:
      - /mnt/tank/configs/journiv:/data
    restart: unless-stopped
    image: swalabtech/journiv-app:latest
```

> `ALLOW_INSECURE_COOKIE_AUTH_OVER_HTTP=true` allows login over plain HTTP on your LAN; behind a reverse proxy, remove it and set `DOMAIN_SCHEME=https`.
> This minimal setup doesn't process media uploads or imports; for those, use the [full upstream compose](https://github.com/journiv/journiv-app/blob/main/docker-compose.yml).
{.is-warning}
