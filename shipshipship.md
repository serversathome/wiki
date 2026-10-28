---
title: ShipShipShip
description: A guide to deploying ShipShipShip
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:08:33.499Z
---

# 🚢 What is ShipShipShip?
A modern, self-hostable changelog and roadmap platform that helps you share product updates with your community and gather feedback through feature voting.

<div class="glance">
  <div><span>Port</span><b><code>8087</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/GauthierNelkinsky/ShipShipShip"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy ShipShipShip

```yaml
services:
  changelog:
    image: nelkinsky/shipshipship:latest
    ports:
      - "8087:8080"
    environment:
      - ADMIN_USERNAME=youradmin
      - ADMIN_PASSWORD=securerpassword
      - JWT_SECRET=your-jwt-secret-change-this # Generate with `openssl rand -hex 32`
      - GIN_MODE=release
      - BASE_URL=https://changelog.yourdomain.com # used for email unsubscribe links; auto-detected if unset
    volumes:
      - /mnt/tank/configs/shipshipship:/app/data
    restart: unless-stopped
```

> Change `ADMIN_USERNAME` and `ADMIN_PASSWORD`, and replace `JWT_SECRET` with your own random value (`openssl rand -hex 32`) before the first start.
{.is-warning}

# 2 · Access ShipShipShip
Right after installation, open the admin panel at `http://IP:8087/admin` and log in with your `ADMIN_USERNAME` / `ADMIN_PASSWORD` to complete setup. The public changelog lives at `http://IP:8087`.
