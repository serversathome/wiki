---
title: Pi-Dash
description: A guide to deploying Pi-Dash
published: true
date: '2026-11-04T16:23:21.253Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:07:12.626Z'
---

# What is Pi-Dash?
Pi-Dash is a simple, lightweight dashboard for monitoring multiple Pi-hole instances. It provides a clean, at-a-glance, responsive view of your Pi-hole statistics.

# :logos:docker-icon: 1 · Deploy Pi-Dash

```yaml
services:
  pi-dash:
    image: ghcr.io/surajverma/pi-dash:latest
    container_name: pi-dash
    ports:
      - 5002:5001
    volumes:
      - /mnt/tank/configs/pidash/config.json:/app/config.json:ro
      # Optional: create manifest.json first (section 1.2), then uncomment
      # - /mnt/tank/configs/pidash/manifest.json:/app/manifest.json:ro
    restart: unless-stopped
```

## 1.1 Config File
> [!NOTE]
> This file needs to be placed in `/mnt/tank/configs/pidash/config.json`

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

> [!NOTE]
> `address` is the base URL of the Pi-hole only, so leave off `/admin` and `/api`. Certificates aren't verified by default; add `"verify_ssl": true` (or a CA bundle path) per instance if your Pi-holes use trusted certs.

## 1.2 Manifest File (Optional)
> [!NOTE]
> Only needed if you want to customize the app manifest. Without it, Pi-Dash uses its built-in manifest. If you use it, place it in `/mnt/tank/configs/pidash/manifest.json` **before** uncommenting the mount, or Docker creates an empty folder in its place.

> [!WARNING]
> Icons on a host with an untrusted self-signed certificate (like `https://pi.hole`) can block the install prompt. Prefer a relative path or a publicly trusted HTTPS URL.
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
