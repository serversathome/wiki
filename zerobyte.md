---
title: Zero Byte
description: A guide to deploying Zerobyte
published: true
date: 2025-12-02T00:24:09.999Z
tags: 
editor: markdown
dateCreated: 2025-12-02T00:17:47.566Z
---

# <img src="/zerobyte.png" class="tab-icon"> What is Zerobyte?

Zerobyte is a backup automation tool that helps you save your data across multiple storage backends. Built on top of Restic, it provides an modern web interface to schedule, manage, and monitor encrypted backups of your remote storage.
Features

- Automated backups with encryption, compression and retention policies powered by Restic
- Flexible scheduling For automated backup jobs with fine-grained retention policies
- End-to-end encryption ensuring your data is always protected
- Multi-protocol support: Backup from NFS, SMB, WebDAV, or local directories

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Zerobyte
```yaml
services:
  zerobyte:
    image: ghcr.io/nicotsx/zerobyte:v0.15
    container_name: zerobyte
    restart: unless-stopped
    cap_add:
      - SYS_ADMIN
    ports:
      - 4096:4096
    devices:
      - /dev/fuse:/dev/fuse
    environment:
      - TZ=America/New_York # Set your timezone here
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ./zerobyte:/var/lib/zerobyte
      - /mnt/tank:/mydata
      - ~/.config/rclone:/root/.config/rclone
```

# 2 · Setting Up rclone
1. To set a new backup target via rclone, run this command as root in the TrueNAS shell and follow the prompts:
```bash
rclone config
```