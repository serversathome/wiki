---
title: Velld
description: A guide to deploying Velld
published: true
date: 2026-01-15T15:32:22.099Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:09:19.943Z
---

# <img src="/velld.png" class="tab-icon"> What is Velld?
A self-hosted database backup management tool. Schedule automated backups, monitor status, and manage multiple databases from one place.

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
```yaml
# Client
NEXT_PUBLIC_API_URL=http://10.99.0.242:8080

# Server
JWT_SECRET=2782e58490cce4ac1f66fe5c1b85ffb0ed3ebfdf233132452c9d16c1c2c9ed3a
ENCRYPTION_KEY=2782e58490cce4ac1f66fe5c1b85ffb0ed3ebfdf233132452c9d16c1c2c9ed3a

# Auth Credentials
ADMIN_USERNAME_CREDENTIAL=admin
ADMIN_PASSWORD_CREDENTIAL=changeme
ALLOW_REGISTER=true

```