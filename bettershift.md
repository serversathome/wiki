---
title: Bettershift
description: A guide to deploying Bettershift
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:03:26.069Z
---

# What is Bettershift?
BetterShift is a modern shift management application designed to simplify variable work schedules. Manage unlimited calendars with one-click shift toggles, reusable presets, and real-time synchronization across devices. Features include external calendar integration (Google, Outlook, iCal), password-protected calendars, ICS/PDF export, live statistics, calendar comparison and multi-language support. Images are amd64 only.

<div class="glance">
  <div><span>Port</span><b><code>3000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/pantelx/bettershift"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Bettershift
```yaml
services:
  bettershift:
    ports:
      - 3000:3000
    volumes:
      - /mnt/tank/configs/bettershift:/app/data
      - /mnt/tank/configs/bettershift/uploads:/app/public/uploads
    environment:
      - BETTER_AUTH_SECRET=CHANGEME
      - BETTER_AUTH_URL=http://your-server-ip:3000
    container_name: bettershift
    image: ghcr.io/pantelx/bettershift:latest
    restart: unless-stopped
```

1. Generate a secret with `openssl rand -hex 32` and paste it in place of `CHANGEME`. Authentication is on by default, and logins fail without a real `BETTER_AUTH_SECRET`
2. Set `BETTER_AUTH_URL` to the address you open BetterShift at, without a trailing slash. Use your public URL instead if you put it behind a reverse proxy or Cloudflare Tunnel
3. Deploy the stack, browse to `http://your-server-ip:3000` and create your account
