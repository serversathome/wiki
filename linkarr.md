---
title: Linkarr
description: A guide to deploying Linkarr
published: true
date: 2025-08-28T14:00:37.397Z
tags: 
editor: markdown
dateCreated: 2025-08-28T14:00:37.397Z
---

# ![](/linkarr.png){class="tab-icon"} What is Linkarr?
Organize your media library with ease - without moving or duplicating your files!

📦 No file moving/copying: Monitors for changes, and then organizes your media with symlinks only.
🧲 Perfect for seeding/usenet: Works with files managed by torrent or usenet clients.
🎬 Jellyfin ready: Import organized folders directly into your media server.
🐳 Easy Docker deployment: Run anywhere, just map your folders.


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Trailarr

```yaml
services:
  my-linkarr:
    image: itsmejoeeey/linkarr:latest
    container_name: linkarr
    volumes:
      - /path/to/source:/path/to/source
      - /path/to/organized:/path/to/organized
      - /mnt/tank/configs/linkarr/config.json:/config/config.json
    restart: unless-stopped
```

## 1.1 Config File
```json
{
  "mode": "watch",
  "media_server_format": "jellyfin",
  "log_level": "info",
  "jobs": [
    {
      "src": "/path/to/source/tv",
      "dest": "/path/to/organized/tv",
      "media_type": "tv"
    },
    {
      "src": "/path/to/source/movies",
      "dest": "/path/to/organized/movies",
      "media_type": "movie",
      "file_type_regex": ".*\\.(mkv|mp4|avi)$"
    }
  ]
} 

```

# 2 · Linkarr Configuration
Ideally you would have two folders for each of your media: `tv` and `tv-organized` for example.

The `/path/to/source` would be your `tv` directory and your `/path/to/organized` would be your `tv-organized` directory. 