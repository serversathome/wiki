---
title: Skysend
description: A guide to deploy Skysend
published: true
date: 2026-04-27T17:59:04.372Z
tags: 
editor: markdown
dateCreated: 2026-04-27T17:59:04.372Z
---

# What is SkySend?

**SkySend** is a minimalist, end-to-end encrypted, self-hostable file and note sharing service. Files and notes are encrypted entirely in the browser using AES-256-GCM before they ever reach the server, so the host stores only encrypted blobs and never has access to the decryption key (which lives only in the URL fragment after the `#`). It supports drag-and-drop file uploads up to 2GB, multi-file uploads (zipped client-side), encrypted notes (text, code snippets with syntax highlighting, Markdown, password sharing, SSH keys), burn-after-reading mode, password protection via Argon2id, and optional S3-compatible storage backends (R2, MinIO, Wasabi, Hetzner, etc.). Inspired by Mozilla Send and PrivateBin, SkySend is a modern, AGPL-licensed alternative built from scratch with no accounts, no telemetry, and no tracking.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy SkySend

```yaml
services:
  skysend:
    image: skyfay/skysend:latest
    container_name: skysend
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - BASE_URL=https://skysend.yourdomain.com
      - TRUST_PROXY=true
      - PUID=568
      - PGID=568
      - FILE_MAX_SIZE=10GB
    volumes:
      - /mnt/tank/configs/skysend/data:/data
      - /mnt/tank/configs/skysend/uploads:/uploads
```


- If you're putting SkySend behind a reverse proxy (Nginx Proxy Manager, Traefik, Cloudflare Tunnel, etc.), set `TRUST_PROXY=true` so it correctly reads `X-Forwarded-For` headers for rate limiting.

 
> Make sure your reverse proxy passes WebSocket upgrades through to SkySend. The WebSocket transport (enabled by default) significantly improves upload speed in Chromium browsers by bypassing HTTP/2 multiplexing bottlenecks.
{.is-warning}

# 2 · Configuration

## 2.1 File Upload Limits

By default, SkySend allows files up to 2GB and 32 files per multi-upload. Adjust these limits via environment variables:

```
environment:
  - FILE_MAX_SIZE=5GB
  - FILE_MAX_FILES_PER_UPLOAD=64
  - FILE_EXPIRE_OPTIONS_SEC=300,3600,86400,604800,2592000
  - FILE_DEFAULT_EXPIRE_SEC=86400
  - FILE_DOWNLOAD_OPTIONS=1,2,3,5,10,20,50,100
  - FILE_DEFAULT_DOWNLOAD=1
```

| Variable | Default | Purpose |
|----------|---------|---------|
| `FILE_MAX_SIZE` | `2GB` | Maximum size per file |
| `FILE_MAX_FILES_PER_UPLOAD` | `32` | Files per multi-upload |
| `FILE_EXPIRE_OPTIONS_SEC` | `300,3600,86400,604800` | Selectable expiry times in seconds |
| `FILE_DEFAULT_EXPIRE_SEC` | `86400` | Default expiry (must match an option) |
| `FILE_UPLOAD_QUOTA_BYTES` | `0` | Per-IP daily upload quota (`0` = unlimited) |


## 2.2 S3 Storage Backend

For larger deployments, point SkySend at S3-compatible object storage instead of local disk. Works with Cloudflare R2, AWS S3, MinIO, Hetzner, Wasabi, Backblaze B2, and others:

```yaml
environment:
  - STORAGE_BACKEND=s3
  - S3_BUCKET=skysend-uploads
  - S3_REGION=auto
  - S3_ENDPOINT=https://<account-id>.r2.cloudflarestorage.com
  - S3_ACCESS_KEY=your-access-key
  - S3_SECRET_KEY=your-secret-key
  - S3_PART_SIZE=25MB
  - S3_CONCURRENCY=4
```

> 
> When using S3 storage, your bucket needs a CORS policy allowing `GET` and `HEAD` from your SkySend domain. Without it, downloads fail with `Access-Control-Allow-Origin` errors.
{.is-warning}

For self-hosted MinIO or Garage, also set `S3_FORCE_PATH_STYLE=true`.

## 2.3 Custom Branding

Rebrand your instance with your own title, color, logo, and footer links:

```yaml
environment:
  - CUSTOM_TITLE=My Secure Share
  - CUSTOM_COLOR=46c89d
  - CUSTOM_LOGO=https://yourdomain.com/logo.svg
  - CUSTOM_PRIVACY=https://yourdomain.com/privacy
  - CUSTOM_LEGAL=https://yourdomain.com/imprint
  - CUSTOM_LINK_URL=https://yourdomain.com
  - CUSTOM_LINK_NAME=Main Site
```

## 2.4 Service Toggles

If you only want file sharing or only notes, control which features are enabled:

```yaml
environment:
  - ENABLED_SERVICES=file,note   # Both (default)
  # ENABLED_SERVICES=file        # Files only
  # ENABLED_SERVICES=note        # Notes only
```

Disabled services return HTTP 403 and their UI tabs are hidden.

