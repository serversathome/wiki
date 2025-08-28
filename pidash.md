---
title: Pi-Dash
description: A guide to deploying Pi-Dash
published: true
date: 2025-08-28T14:38:41.759Z
tags: 
editor: markdown
dateCreated: 2025-08-28T14:38:41.759Z
---

# ![](/pi-hole.png){class="tab-icon"} What is Pi-Dash?
Pi-Dash is a simple, lightweight dashboard for monitoring multiple Pi-hole instances. It provides a clean, at-a-glance, responsive view of your Pi-hole statistics.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Pi-Dash

```yaml
services:
  pi-dash:
    image: ghcr.io/surajverma/pi-dash:latest
    container_name: pi-dash
    ports:
      - 5002:5001
    volumes:
      - /mnt/tank/configs/pidash/config.json:/app/config.json
      - /mnt/tank/configs/pidash/manifest.json:/app/manifest.json
```

## 1.1 Config File
> This file needs to be placed in `/mnt/tank/configs/pidash`
{.is-info}

```json
{
  "refresh_interval": 1000,
  "piholes": [
    {
      "name": "Primary",
      "address": "https://pi.hole/one",
      "password": "your_app_password_here",
      "enabled": true
    },
    {
      "name": "Secondary",
      "address": "https://pi.hole/two",
      "password": "your_app_password_here",
      "enabled": true
    }
  ]
}
```

## 1.2 Manifest File
> This file needs to be placed in `/mnt/tank/configs/pidash`
{.is-info}
```json
{
  "name": "Pi-Dash",
  "short_name": "Pi-Dash",
  "description": "A simple dashboard to monitor Pi-hole status.",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#111827",
  "theme_color": "#06b6d4",
  "icons": [
    {
      "src": "https://pi.hole/admin/img/logo.svg",
      "sizes": "512x512",
      "type": "image/svg+xml"
    }
  ]
}
```