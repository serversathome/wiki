---
title: ShipShipShip
description: A guide to deploying ShipShipShip
published: true
date: 2025-08-28T14:26:28.770Z
tags: 
editor: markdown
dateCreated: 2025-08-28T14:26:28.770Z
---

# 🚢 What is ShipShipShip?
A modern, self-hostable changelog and roadmap platform that helps you share product updates with your community and gather feedback through feature voting.

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
      - JWT_SECRET=your-jwt-secret-change-this
      - GIN_MODE=release
    volumes:
      - /mnt/tank/configs/shipshipship:/app/data
    restart: unless-stopped
```
