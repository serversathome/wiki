---
title: Velld
description: A guide to deploying Velld
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:09:19.943Z
---

# <img src="/velld.png" class="tab-icon"> What is Velld?
A self-hosted database backup management tool. Schedule automated backups, monitor status, and manage multiple databases from one place.

<div class="glance">
  <div><span>Port</span><b><code>3000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>2 services</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/dendianugerah/velld"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Velld

```yaml
services:
  api:
    image: ghcr.io/dendianugerah/velld/api:latest
    ports:
      - 8080:8080
    env_file:
      - .env
    volumes:
      - /mnt/tank/configs/velld/api_data:/app/data
      - /mnt/tank/configs/velld/backup_data:/app/backups
    restart: unless-stopped
  web:
    image: ghcr.io/dendianugerah/velld/web:latest
    ports:
      - 3000:3000
    environment:
      NEXT_PUBLIC_API_URL: ${NEXT_PUBLIC_API_URL}
      ALLOW_REGISTER: ${ALLOW_REGISTER}
    depends_on:
      - api
    restart: unless-stopped
```

## env File
```env
# Client
NEXT_PUBLIC_API_URL=http://YOUR-SERVER-IP:8080

# Server
JWT_SECRET=CHANGE_ME_64_CHAR_HEX
ENCRYPTION_KEY=CHANGE_ME_DIFFERENT_64_CHAR_HEX

# Auth Credentials
ADMIN_USERNAME_CREDENTIAL=admin
ADMIN_PASSWORD_CREDENTIAL=changeme
ALLOW_REGISTER=true

```

1. Run this twice and paste a **different** value into `JWT_SECRET` and `ENCRYPTION_KEY`:
   ```bash
   openssl rand -hex 32
   ```
1. Set `NEXT_PUBLIC_API_URL` to your server's IP, change the admin username and password, and deploy.
1. Open the web UI at `http://YOUR-SERVER-IP:3000` and log in with your admin credentials. Port `8080` is the API, not the web UI.
