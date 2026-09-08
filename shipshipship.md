---
title: ShipShipShip
description: A guide to deploying ShipShipShip
published: true
date: 2026-01-15T15:31:48.178Z
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
      - JWT_SECRET=your-jwt-secret-change-this
      - GIN_MODE=release
    volumes:
      - /mnt/tank/configs/shipshipship:/app/data
    restart: unless-stopped
```
