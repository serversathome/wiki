---
title: Cross Seed
description: A guide on how to deploy Cross Seed
published: true
date: 2025-06-11T09:49:16.194Z
tags: 
editor: markdown
dateCreated: 2025-06-11T09:31:37.247Z
---

![cross-seed.png](/cross-seed.png)


# What is Cross Seed?
cross-seed is an app designed to help you download torrents that you can cross seed based on your existing torrents. It is designed to match conservatively to minimize manual intervention. cross-seed can inject the torrents it finds directly into your torrent client. 

# Docker Compose
```yaml
services:
  cross-seed:
    image: ghcr.io/cross-seed/cross-seed:6
    container_name: cross-seed
    user: 568:568 # this must match your torrent client (cross-seed does not support using PGID and PUID)
    ports:
      - "2468:2468"
    volumes:
      - /mnt/tank/configs/crossseed:/config
      - /mnt/tank/media:/media
    command: daemon
    restart: unless-stopped
```

# Configuration
1. Navigate to the `configs` folder and open the `config.js` file
1. E
1. Start the daemon by running this command in the TrueNAS Shell as `root`:
```bash
docker exec -it cross-seed cross-seed daemon
```


# Troubleshooting
The Cross Seed docs are excellent. Follow their instructions [here](https://www.cross-seed.org/docs/basics/faq-troubleshooting).

# Video