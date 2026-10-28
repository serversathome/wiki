---
title: PruneMate
description: A guide to deploying PruneMate
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:33.318Z
---

# <img src="/prunemate.png" class="tab-icon"> What is PruneMate?
A sleek, lightweight web interface to automatically clean up Docker resources on a schedule.

<div class="glance">
  <div><span>Port</span><b><code>7676</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy PruneMate
```yaml
services:
  prunemate:
    image: anoniemerd/prunemate:latest
    container_name: prunemate
    ports:
      - "7676:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /mnt/tank/configs/prunemate/logs:/var/log
      - /mnt/tank/configs/prunemate:/config
    environment:
      - PRUNEMATE_TZ=America/New_York # Change this to your desired timezone
      - PRUNEMATE_TIME_24H=true #false for 12-Hour format (AM/PM)
      # Optional: Enable authentication (generate hash with: docker run --rm anoniemerd/prunemate:latest python prunemate.py --gen-hash "password")
      # - PRUNEMATE_AUTH_USER=admin
      # - PRUNEMATE_AUTH_PASSWORD_HASH=your_base64_encoded_hash_here
    restart: unless-stopped
```

> PruneMate has the Docker socket and can delete images, containers, volumes and networks. Turn on authentication by generating a password hash with the command in the compose comment and uncommenting the two `PRUNEMATE_AUTH_*` lines.
{.is-warning}