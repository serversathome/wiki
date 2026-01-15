---
title: Emby Wrapped
description: A guide to deploying Emby Wrapped
published: true
date: 2026-01-15T15:29:08.184Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:33.448Z
---

# What is Emby Wrapped?

A beautiful, Spotify Wrapped-style year-in-review experience for your Emby media server. See your viewing stats, top shows, favorite genres, and more in an interactive, animated presentation.
# <img src="/docker.png" class="tab-icon"> 1 · Deploy 
```yaml
services:
  emby-wrapped:
    image: ghcr.io/davidtorcivia/emby-wrapped-ftp:latest
    container_name: emby-wrapped
    ports:
      - "3000:3000"
    environment:
      - EMBY_URL=http://your-emby-server:8096
      - EMBY_API_KEY=your-api-key-here
      - TMDB_API_KEY=
    restart: unless-stopped
```