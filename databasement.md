---
title: Databasement
description: A guide to deploying databasement
published: true
date: 2026-02-03T15:45:15.644Z
tags: 
editor: markdown
dateCreated: 2026-02-03T15:45:15.644Z
---

# <img src="/databasement.png" class="tab-icon"> What is Databasement?

**Databasement** is a web-based database backup management application for MySQL, PostgreSQL, and MariaDB. It allows you to schedule automated backups, store them locally or on S3-compatible storage, and restore snapshots to any registered server — including cross-server restores (e.g., production to staging).

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Databasement

```yaml
services:
  databasement:
    image: davidcrty/databasement:latest
    container_name: databasement
    environment:
      - APP_KEY=base64:GENERATE_THIS_KEY
      - DB_CONNECTION=sqlite
      - DB_DATABASE=/data/database.sqlite
      - ENABLE_QUEUE_WORKER=true
      - TZ=America/New_York
      - PUID=1000
      - PGID=1000
    restart: unless-stopped
    ports:
      - "2226:2226"
    volumes:
      - /mnt/tank/configs/databasement:/data
```

1. Generate an APP_KEY:
   ```bash
   docker run --rm davidcrty/databasement:latest php artisan key:generate --show
   ```
2. Replace `APP_KEY=base64:GENERATE_THIS_KEY` with the output

> 
> The `ENABLE_QUEUE_WORKER=true` is required for processing backup and restore jobs. Without it, jobs will be queued but never executed.
{.is-warning}

# 2 · Configuration

## 2.1 Adding Database Servers

After logging in, navigate to **Database Servers** and add your MySQL, PostgreSQL, or MariaDB servers. Databasement connects to them remotely to perform backups.

> 
> If your database is running in Docker on the same host, use `host.docker.internal` or the container name (if on the same Docker network) instead of `localhost`.
{.is-info}

## 2.2 Storage Volumes

By default, backups are stored in the `/data` volume. You can also configure S3-compatible storage (AWS S3, MinIO, Wasabi, etc.) or SFTP/FTP destinations from the web UI.

## 2.3 Scheduling Backups

Create backup schedules for daily, weekly, or monthly backups with customizable retention policies. Databasement automatically prunes old snapshots based on your retention settings.

## 2.4 Notifications

Configure failure notifications via:
- Email (SMTP)
- Slack webhooks
- Discord webhooks

## 2.5 External Database (Optional)

For production deployments, you can use an external PostgreSQL or MySQL database instead of SQLite:

```yaml
environment:
  - DB_CONNECTION=pgsql
  - DB_HOST=your-postgres-host
  - DB_PORT=5432
  - DB_DATABASE=databasement
  - DB_USERNAME=databasement
  - DB_PASSWORD=your-password
```

| Environment Variable | Description |
|---------------------|-------------|
| `APP_KEY` | Laravel encryption key (required) |
| `DB_CONNECTION` | `sqlite`, `pgsql`, or `mysql` |
| `DB_DATABASE` | Database path (SQLite) or name |
| `ENABLE_QUEUE_WORKER` | Enable background job processing |
| `PUID` / `PGID` | User/group ID for file permissions |
| `TZ` | Timezone for schedules |
{.dense}

# <img src="/youtube.png" class="tab-icon"> 3 · Video

*No video yet — check back soon!*