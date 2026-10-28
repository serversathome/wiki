---
title: Zero Byte
description: A guide to deploying Zerobyte
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:09:47.387Z
---

# <img src="/zerobyte.png" class="tab-icon"> What is Zerobyte?

Zerobyte is a backup automation tool that helps you save your data across multiple storage backends. Built on top of Restic, it provides an modern web interface to schedule, manage, and monitor encrypted backups of your remote storage.
- Automated backups with encryption, compression and retention policies powered by Restic
- Flexible scheduling For automated backup jobs with fine-grained retention policies
- End-to-end encryption ensuring your data is always protected
- Multi-protocol support: Backup from NFS, SMB, WebDAV, or local directories

<div class="glance">
  <div><span>Port</span><b><code>4096</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/nicotsx/zerobyte"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Zerobyte
```yaml
services:
  zerobyte:
    image: ghcr.io/nicotsx/zerobyte:v0.43
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
      - BASE_URL=http://YOUR-SERVER-IP:4096 # URL you will use to access Zerobyte (use https:// for secure cookies)
      - APP_SECRET=CHANGE_ME # Generate your own secret with: openssl rand -hex 32
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /mnt/tank/configs/zerobyte:/var/lib/zerobyte
      - /mnt/tank:/mydata
      - ~/.config/rclone:/root/.config/rclone:ro
```

1. `BASE_URL` and `APP_SECRET` are **required**. Set `BASE_URL` to the URL you open Zerobyte at, and generate `APP_SECRET` (32+ characters, used to encrypt sensitive data in the database):
   ```bash
   openssl rand -hex 32
   ```
1. If you only back up locally mounted folders (no NFS/SMB/WebDAV remote mounts), you can remove the `cap_add: SYS_ADMIN` and `/dev/fuse` device lines.

> Don't expose Zerobyte to the internet. If the server is reachable from outside (VPS or port forwarding), change the port mapping to `127.0.0.1:4096:4096` and use a secure tunnel with authentication.
{.is-warning}

> **Upgrading from an older version?** Older versions of this guide used the `v0.15` tag, which is no longer published. Newer releases require `BASE_URL` and `APP_SECRET`. Back up `/mnt/tank/configs/zerobyte` before switching to `v0.43`, and check the [release notes](https://github.com/nicotsx/zerobyte/releases) for migration steps.
{.is-info}

# 2 · Setting Up rclone
1. To set a new backup target via rclone, run this command as root in the TrueNAS shell and follow the prompts:
```bash
rclone config
```
