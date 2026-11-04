---
title: ShipShipShip
description: A guide to deploying ShipShipShip
published: true
date: '2026-11-04T16:23:45.592Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:08:33.499Z'
---

# 🚢 What is ShipShipShip?
A modern, self-hostable changelog and roadmap platform that helps you share product updates with your community and gather feedback through feature voting.

# :logos:docker-icon: 1 · Deploy ShipShipShip

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

> [!WARNING]
> Change `ADMIN_USERNAME` and `ADMIN_PASSWORD`, and replace `JWT_SECRET` with your own random value (`openssl rand -hex 32`) before the first start.

# 2 · Access ShipShipShip
Right after installation, open the admin panel at `http://IP:8087/admin` and log in with your `ADMIN_USERNAME` / `ADMIN_PASSWORD` to complete setup. The public changelog lives at `http://IP:8087`.

