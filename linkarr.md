---
title: Linkarr
description: A guide to deploying Linkarr
published: true
date: 2025-08-28T14:07:02.946Z
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

<table>
<thead>
<tr>
<th></th>
<th>📂 <strong>Before (Source Folder)</strong></th>
<th>📂 <strong>After (Organized Folder)</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td>TV</td>
<td><code>/media/TV/</code><br>└─ 📂<code>Show.Name.S01/</code><br>&nbsp;&nbsp;&nbsp;&nbsp;├── <code>Show.Name.S01E01.1080p.mkv</code><br>&nbsp;&nbsp;&nbsp;&nbsp;└── <code>Show.Name.S01E02.1080p.mkv</code><br>└─📂 <code>Show.Name.S02/</code><br>&nbsp;&nbsp;&nbsp;&nbsp;├── <code>Show.Name.S02E01.1080p.mkv</code><br>&nbsp;&nbsp;&nbsp;&nbsp;└── <code>Show.Name.S02E02.1080p.mkv</code><br>└── <code>Another.Show.Name.S04E07.1080p.mkv</code></td>
<td><code>/organized/TV/</code><br>└─🗂️ <code>Show Name/</code><br>&nbsp;&nbsp;&nbsp;&nbsp;└─📂 <code>Season 01/</code><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├─🔗 <code>Show.Name.S01E01.1080p.mkv</code><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;└─🔗 <code>Show.Name.S01E02.1080p.mkv</code> <br>&nbsp;&nbsp;&nbsp;&nbsp;└─📂 <code>Season 02/</code><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├─🔗 <code>Show.Name.S02E01.1080p.mkv</code><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;└─🔗 <code>Show.Name.S02E02.1080p.mkv</code> <br>└─🗂️ <code>Another Show Name/</code><br>&nbsp;&nbsp;&nbsp;&nbsp;└─📂 <code>Season 04/</code><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;└─🔗 <code>Show.Name.S04E07.1080p.mkv</code></td>
</tr>
<tr>
<td>Movies</td>
<td><code>/media/Movies/</code><br>└─📂 <code>Movie.Title.2023.1080p/</code><br>&nbsp;&nbsp;&nbsp;&nbsp;└── <code>Movie.Title.2023.1080p.mkv</code><br>└── <code>Another.Movie.Title.2024.1080p.mkv</code></td>
<td><code>/organized/Movies/</code><br>└─📂 <code>Movie Title (2023)/</code><br>&nbsp;&nbsp;&nbsp;&nbsp;└─🔗 <code>Movie.Title.2023.1080p.mkv</code> <br>└─📂 <code>Another Movie Title (2024)/</code><br>&nbsp;&nbsp;&nbsp;&nbsp;└─🔗 <code>Another.Movie.Title.2024.1080p.mkv</code></td>
</tr>
</tbody>
</table>


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
> You must map your **source** and **destination** folders to the same paths inside the container as on your host system. This ensures symlinks work correctly.
{.is-warning}


## 1.1 Config File
> Place this file in `/mnt/tank/configs/linkarr/`
{.is-info}

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