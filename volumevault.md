---
title: VolumeVault
description: A guide to deploying VolumeVault
published: true
date: 2026-06-02T20:33:22.276Z
tags: 
editor: markdown
dateCreated: 2026-05-30T11:20:56.597Z
---

# <img src="/volumevault.png" class="tab-icon"> What is VolumeVault?

**VolumeVault** is a self-hosted web UI for backing up and restoring your Docker volumes and host paths. It's a Laravel app that wraps the well-regarded [`offen/docker-volume-backup`](https://github.com/offen/docker-volume-backup) engine, giving you a clean dashboard for scheduled jobs, encrypted off-site destinations, restore runs, notifications, and run history — instead of hand-writing backup labels and cron entries for every stack.

It does **not** replace the backup engine; it orchestrates it. VolumeVault discovers your volumes through the Docker socket, shows you backup coverage per volume or per Compose/Swarm stack, and drives offen's tooling to actually move the data.

What you get:

- **Volume & host-path discovery** — finds your Docker volumes and surfaces backup coverage by volume or by stack, with the last known backup size where available
- **Lots of destinations** — AWS S3, Cloudflare R2, any S3-compatible store, WebDAV, SSH/SFTP, Azure Blob, Dropbox, Google Drive, and plain local filesystem
- **Encrypted secrets at rest** — destination credentials and notification URLs are encrypted with Laravel's `Crypt` (which is why your `APP_KEY` matters — see below)
- **Flexible schedules** — hourly, daily, weekly, or full cron expressions; plus manual runs, pause/resume, and log inspection
- **Safer restores** — archives restore into *new* Docker volumes by default rather than clobbering the originals
- **Notifications** — per-job Shoutrrr channels (Discord, Telegram, email, etc.) with a default channel for new jobs
- **API & automation** — Sanctum API tokens for scripts, dashboards, and agents
- **Portable installation saves** — export an encrypted save and import it during onboarding to move or rebuild an instance




# <img src="/docker.png" class="tab-icon"> 1 · Deploy VolumeVault

## 1.1 · Generate an App Key

VolumeVault encrypts your stored credentials with a Laravel `APP_KEY`, so you generate one first and paste it into your compose file. Run this one-off command (it just prints a key, runs nothing persistent):

```bash
docker run --rm ghcr.io/darkdragon14/volumevault:v1.6.0 php artisan key:generate --show
```

Copy the full output, including the `base64:` prefix.


> Keep this `APP_KEY` safe and back it up. It's required to decrypt your destinations, notification URLs, and installation saves — lose it and you lose access to those stored secrets. Pin a version tag in production rather than `latest`.
{.is-warning}

## 1.2 · Compose

```yaml
services:
  volumevault:
    image: ghcr.io/darkdragon14/volumevault:v1.6.0
    container_name: volumevault
    ports:
      - "8081:8080"
    environment:
      APP_KEY: 
      APP_URL: 
      APP_TIMEZONE: America/New_York
      # Restrict which host paths can be backed up (recommended):
      VOLUMEVAULT_HOST_PATH_ALLOWLIST: /mnt/tank
    volumes:
      - /mnt/tank/configs/volumevault:/app/storage
      - /var/run/docker.sock:/var/run/docker.sock
    restart: unless-stopped
```


## 1.3 · Host Path Allowlist

If you plan to back up **host paths** (not just named Docker volumes), set `VOLUMEVAULT_HOST_PATH_ALLOWLIST` to a comma-separated list of permitted path prefixes, e.g. `/mnt/tank,/srv`. When set, any host-path job pointing outside those prefixes is rejected on save — a good guardrail given the socket access. Leave it unset if you only back up named volumes.

# 2 · Configuration

## 2.1 First Login & Onboarding

After the container is up, the onboarding screen walks you through creating the admin account. From there you can either start fresh or **import an installation save** (an encrypted export from another VolumeVault instance) to restore your jobs and destinations.

## 2.2 Destinations

Add one or more backup destinations under the destinations section. Supported backends include AWS S3, Cloudflare R2, generic S3-compatible storage, WebDAV, SSH/SFTP, Azure Blob, Dropbox, Google Drive, and the local filesystem. Credentials are encrypted at rest with your `APP_KEY`. For a homelab, an S3-compatible target (your own MinIO, Backblaze B2 via S3, or R2) or an SFTP box makes a solid off-host destination.

> 
> Follow 3-2-1: a local filesystem destination on the same host is convenient but isn't a real backup if that host dies. Pair it with at least one off-host/off-site destination (S3-compatible or SFTP).
{.is-success}

## 2.3 Backup Jobs & Schedules

Create jobs against discovered volumes or allowed host paths, and schedule them hourly, daily, weekly, or with a cron expression. You can run jobs manually, pause/resume them, and inspect per-run logs and history. Coverage is shown per volume and per stack so you can spot anything that *isn't* being backed up.

## 2.4 Restores

Restores are deliberately conservative: by default an archive is restored into a **new** Docker volume rather than overwriting the original, so a restore can't silently destroy live data. Verify the new volume, then point your service at it.

## 2.5 Notifications

Each job can post to a Shoutrrr notification channel (Discord, Telegram, email, Slack, and many more via Shoutrrr URL syntax). Set a default channel for new jobs, and backup size is included in messages when known. To use email notifications, configure the `MAIL_*` environment variables.

## 2.6 API Tokens

VolumeVault issues Sanctum API tokens for automation — trigger backups, read job state, or feed a dashboard. Remember the security note: a write-capable token is as powerful as host access, so scope and store tokens carefully.

## 2.7 Reverse Proxy

Set `APP_URL` to your public HTTPS URL and front the container with your usual proxy (Cloudflare Tunnel or Nginx pointing at `:8080`). If your proxy sits in front, you may also need `TRUSTED_PROXIES` set so Laravel honors the forwarded scheme/host. Given the socket access, keep this behind authentication and avoid exposing it publicly without a good reason.



https://youtu.be/PLACEHOLDER