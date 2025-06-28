---
title: Cleanuparr
description: A guide to deploying Cleanuparr via docker
published: true
date: 2025-06-28T13:01:32.166Z
tags: 
editor: markdown
dateCreated: 2025-06-28T12:56:06.603Z
---

![cleanuparr.png](/cleanuparr.png)

# What is Cleanuparr?

Automated Download Management. Automatically clean up unwanted, stalled, and malicious downloads from your \*arr applications and download clients. Keep your queues clean and your media library safe.

# Installation

> Read the [official documentation](https://cleanuparr.github.io/Cleanuparr/docs/)
{.is-info}


```yaml
services:
  cleanuparr:
    image: ghcr.io/cleanuparr/cleanuparr:latest
    container_name: cleanuparr
    restart: unless-stopped
    ports:
      - 11011:11011
    volumes:
      - /mnt/tank/configs/cleanuparr:/config
      - /mnt/tank/media:/media
    environment:
      - PORT=11011
      - BASE_PATH=
      - PUID=568
      - PGID=568
      - UMASK=022
      - TZ=America/New_York
```

## Permissions & Folder Structure

- **PUID / PGID**: Ensure you use a user/group with the correct permissions for accessing media folders. TrueNAS SCALE defaults to 568:568 for apps.
- **Volumes**: The container structure follows a common-sense naming convention, storing configurations under /mnt/tank/configs/radarr
- 📌 Refer to the [Folder-Structure](/Folder-Structure) guide for more details.