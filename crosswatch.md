---
title: CrossWatch
description: A guide to deploying CrossWatch
published: true
date: 2026-01-15T15:28:49.743Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:10.399Z
---

# <img src="/crosswatch.png" class="tab-icon"> What is CrossWatch?
Synchronize your data across Plex, Jellyfin, SIMKL, Trakt, and more. Keep your movies and shows in sync, no matter where you watch. 

<div class="glance">
  <div><span>Port</span><b><code>8787</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy CrossWatch

```yaml
services:
  crosswatch:
    image: ghcr.io/cenodude/crosswatch:latest
    container_name: crosswatch
    ports:
      - 8787:8787
    environment:
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/crosswatch:/config
    restart: unless-stopped