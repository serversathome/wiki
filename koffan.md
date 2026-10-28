---
title: Koffan
description: A guide to deploying Koffan
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:05:48.703Z
---

# What is Koffan?
Koffan is a lightweight web application for managing shopping lists, designed for couples and families. It allows real-time synchronization between multiple devices, so everyone knows what to buy and what's already in the cart.

The app works in any browser on both mobile and desktop. Just one password to log in - no complicated registration required.

<div class="glance">
  <div><span>Port</span><b><code>3000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/PanSalut/Koffan"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Koffan

> **Upgrading an existing install?** Since 2.10.0 the container listens on port `8080` instead of `80`, so update your port mapping (and any reverse proxy upstream) to `8080`. Version 2.16.0 also runs a one-way database migration on first start, so back up `/mnt/tank/configs/koffan/shopping.db` before pulling. There's no downgrade path. See the [upstream upgrade notes](https://github.com/PanSalut/Koffan#upgrading).
{.is-warning}

```yaml
services:
  koffan:
    ports:
      - 3000:8080
    environment:
      - APP_ENV=production
      - APP_PASSWORD=CHANGE_ME # generate with: openssl rand -base64 24
    volumes:
      - /mnt/tank/configs/koffan:/data
    image: ghcr.io/pansalut/koffan:latest
    restart: unless-stopped
    container_name: koffan
```

Replace `CHANGE_ME` with the shared password you'll use to log in.
