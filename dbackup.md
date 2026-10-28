---
title: DBackup
description: A guide to deploying DBackup
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-04-27T17:49:27.607Z
---

# What is DBackup?

**DBackup** is a self-hosted database backup automation tool that handles scheduled dumps, encryption, compression, and offsite uploads through a clean web UI. It supports MySQL, MariaDB, PostgreSQL, MongoDB, Redis, SQLite, and Microsoft SQL Server, and can ship those backups to over a dozen destinations including local disk, Amazon S3, Cloudflare R2, Hetzner Object Storage, MinIO, SFTP, FTP/FTPS, WebDAV, SMB, Rsync, Google Drive, Dropbox, and OneDrive.

Backups are encrypted at rest with AES-256-GCM, compressed with GZIP or Brotli, and rotated using Grandfather-Father-Son retention. The web interface gives you live job progress, restore-with-remap, multi-destination jobs for redundancy, and notifications via Discord, Slack, Teams, Telegram, Gotify, ntfy, Twilio SMS, generic webhooks, or SMTP email. SSO/OIDC and RBAC are baked in for team use.

<div class="glance">
  <div><span>Port</span><b><code>3000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
</div>



# <img src="/docker.png" class="tab-icon"> 1 · Deploy DBackup

> **Upgrading from v3 to v4:** stop the container and back up `/mnt/tank/configs/dbackup/data` (at least `data/db`) first. On first start, leave the container running until the log shows `All migrations have been successfully applied`; stopping it mid-migration can block the next start. Retention now follows renamed jobs and may delete old backups in one go, so lock any you want to keep. Configuration backups now require an encryption key (**Settings → Configuration backup**). See the [release notes](https://github.com/Skyfay/DBackup/releases).
{.is-warning}

```yaml
services:
  dbackup:
    image: skyfay/dbackup:latest
    container_name: dbackup
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - ENCRYPTION_KEY=          # openssl rand -hex 32
      - BETTER_AUTH_SECRET=      # openssl rand -base64 32
      - BETTER_AUTH_URL=https://dbackup.example.com
      - TZ=America/New_York
      - PUID=1000
      - PGID=1000
      # - DISABLE_HTTPS=true     # Optional: serve plain HTTP (use with reverse proxy)
      # - TRUSTED_ORIGINS=http://192.168.1.10:3000
      # - TRUSTED_PROXIES=private   # Optional: trust X-Forwarded-For from reverse proxies on private networks (or `cloudflare`)
    volumes:
      - /mnt/tank/configs/dbackup/data:/data
      - /mnt/tank/configs/dbackup/backups:/backups
      - /mnt/tank/configs/dbackup/tmp:/tmp   # Recommended: staging space so running backups don't fill the Docker disk
    stop_grace_period: 10m
```




> 
> The `stop_grace_period: 10m` line matters. By default Docker sends `SIGKILL` 10 seconds after `docker stop`, which will hard-kill an in-progress backup. DBackup catches `SIGTERM` and waits for running jobs to finish, but only if Docker actually waits — so give it real time to drain.
{.is-info}

> 
> If you want to put DBackup behind Nginx Proxy Manager, Pangolin, or another reverse proxy, set `DISABLE_HTTPS=true` and let the proxy terminate TLS. Otherwise you will be double-wrapping HTTPS, which usually breaks WebSocket-based live progress. Also set `TRUSTED_PROXIES` (e.g. `private` or `cloudflare`) so DBackup sees the real client IP for logins and rate limits.
{.is-success}

# 2 · Configuration

## 2.1 First Login & Admin Setup

The first account you create is the admin. After login:

1. Go to **Settings → Profile** and enable two-factor authentication.
2. Open **Vault** in the sidebar, switch to the **Encryption** tab, and click **Recovery kit** to download it. Store the recovery kit and the `ENCRYPTION_KEY` in two different places (e.g., 1Password and an offline copy on a USB drive in a fire safe). The recovery kit is what saves you if the container's database is corrupted but the encrypted backups still exist.
3. If you want SSO, head to **Users & Groups → SSO**, click **New provider**, and wire up Authentik, Authelia, or your IdP of choice.

## 2.2 Add a Database Source

1. Go to **Connections → Databases** and click **New database**.
2. Pick the engine (MySQL, PostgreSQL, MongoDB, Redis, SQLite, or MSSQL).
3. Enter host, port, credentials, and database name. For MySQL/PostgreSQL you can use a read-only backup user with `SELECT`, `LOCK TABLES`, `SHOW VIEW`, `EVENT`, and `TRIGGER` (MySQL) or `pg_dump`-equivalent privileges (Postgres).
4. Click **Test connection**, then **Create database**.

| Source | Notes |
|--------|-------|
| MySQL / MariaDB | Uses `mysqldump`. Supports SSL and `--single-transaction` for InnoDB. |
| PostgreSQL | Uses `pg_dump`. Versions 12–18 supported. |
| MongoDB | Uses `mongodump`. Supports replica sets and Atlas. |
| Redis | Snapshots `dump.rdb` over the wire. |
| SQLite | Local file path or SSH-mounted file. |
| Microsoft SQL Server | 2017, 2019, 2022. Uses native backup format. |


## 2.3 Add a Storage Destination

Under **Connections → Destinations → New destination**, common picks for homelabs:

- **Local Filesystem** → writes to `/backups` inside the container (`/mnt/tank/configs/dbackup/backups` on the host). Useful for fast, local restores.
- **S3 Compatible** → point at MinIO, Wasabi, Backblaze B2 (S3 API), or any other S3 endpoint.
- **SFTP** → SSH into another box on your network, including TrueNAS itself. Supports password, private key, and SSH agent auth.
- **SMB / Samba** → write to a Windows/TrueNAS SMB share.
- **WebDAV** → great for Nextcloud or ownCloud/OpenCloud targets.

A single backup job can target multiple destinations at once, so you can write locally for speed and to S3 for offsite without running two jobs.

## 2.4 Create a Backup Job

1. Go to **Jobs** and click **New job**.
2. Pick the source database and one or more destinations.
3. Set the **schedule** with a cron expression (e.g., `0 3 * * *` for 3 AM daily).
4. Configure **retention**: either a flat "keep N days" or full GFS (e.g., 7 daily / 4 weekly / 12 monthly / 5 yearly).
5. Enable **encryption** and **compression** (Brotli compresses better, GZIP is faster).
6. Attach a notification channel so you actually find out when something fails.
7. Click **Create job**, then use **Run now** on the job's row once and check the **Backups** page to confirm everything works end-to-end before trusting the schedule.

## 2.5 Restore a Backup

From **Backups** in the sidebar, pick a backup, click **Restore**, and choose the target server and databases (or use **History → Restore from this backup**). DBackup verifies the checksum, decrypts in a temp dir, and restores. You can also restore to a *different* database name on the same server — handy for testing restores without nuking production.






