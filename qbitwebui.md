---
title: qBitwebui
description: A guide to deploying qbitwebui
published: true
date: 2026-02-03T13:43:28.569Z
tags: 
editor: markdown
dateCreated: 2026-02-03T13:43:28.569Z
---

# <img src="/qbitwebui.png" class="tab-icon"> What is qbitwebui?

**qbitwebui** is a modern, lightweight web interface for qBittorrent built with React and Vite. It connects to your existing qBittorrent instance and provides a fresh, responsive UI with features like real-time monitoring, multi-instance support, Prowlarr integration, and cross-seed detection.

**Key Features:**
- Real-time torrent monitoring with auto-refresh
- Multi-instance support — manage multiple qBittorrent servers from one dashboard
- Prowlarr integration for searching indexers directly
- Cross-seed detection to find cross-seeding opportunities
- Network agent with speedtest, IP check, and diagnostics
- File browser for managing downloaded files
- Filter by status, tracker, or category
- Multi-select with bulk actions (start, stop, delete)
- Built-in themes with custom theme editor

# <img src="/docker.png" class="tab-icon"> 1 · Deploy qbitwebui

```yaml
services:
  qbitwebui:
    image: ghcr.io/maciejonos/qbitwebui:latest
    container_name: qbitwebui
    environment:
      - ENCRYPTION_KEY=fba1a51c6e7e8fdb216a5f6f3e3a456d7cf80ab5e6ab1a0f1a86f1d7e7a1f77bc
      # Uncomment to disable login (single-user mode)
      # - DISABLE_AUTH=true
      # Uncomment to disable registration (creates default admin account)
      # - DISABLE_REGISTRATION=true
      # Uncomment to allow HTTPS with self-signed certificates
      # - ALLOW_SELF_SIGNED_CERTS=true
      - DOWNLOADS_PATH=/media/downloads
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - /mnt/tank/configs/qbitwebui:/data
      - /mnt/tank/media:/media
```

# 2 · Configuration

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `ENCRYPTION_KEY` | Secret key for encrypting stored credentials (generate with `openssl rand -hex 32`) | Yes |
| `DISABLE_AUTH` | Set to `true` to disable login (single-user mode) | No |
| `DISABLE_REGISTRATION` | Set to `true` to disable registration (creates default admin account) | No |
| `ALLOW_SELF_SIGNED_CERTS` | Set to `true` to allow connecting to qBittorrent instances with self-signed HTTPS certificates | No |
| `DOWNLOADS_PATH` | Set to `/downloads` to enable the file browser feature | No |
{.dense}

## Adding qBittorrent Instances

After deploying qbitwebui:

1. Navigate to `http://your-server-ip:3000`
2. Create an account (or use the default admin if registration is disabled)
3. Add your qBittorrent instance(s) through the web interface
4. Enter the URL and credentials for each qBittorrent server


# 3 · Resources

- **GitHub:** https://github.com/Maciejonos/qbitwebui
- **Documentation:** https://maciejonos.github.io/qbitwebui/

# <img src="/youtube.png" class="tab-icon"> 4 · Video

*No video yet — check back soon!*