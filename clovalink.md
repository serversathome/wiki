---
title: ClovaLink
description: A guide to deploying ClovaLink
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-02-03T14:11:22.216Z
---

# <img src="/clovalink.png" class="tab-icon"> What is ClovaLink?

**ClovaLink** is a Rust-powered, multi-tenant file management platform with HIPAA/SOX/GDPR compliance modes. It supports pluggable storage backends (S3, Wasabi, Backblaze B2, MinIO, or local volumes) and includes built-in virus scanning via ClamAV.

Think of it as a self-hosted alternative to Box or Dropbox Business — enterprise file management without the per-user pricing.

<div class="glance">
  <div><span>Port</span><b><code>3000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>5 services</b></div>
  <div><span>Depends on</span><b>Postgres and Redis</b></div>
  <div><span>Difficulty</span><b class="difficulty intermediate">Intermediate</b></div>
  <div class="glance-links">
    <a href="https://github.com/ClovaLink/ClovaLink"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy ClovaLink

```yaml
services:
  clovalink-backend:
    image: ghcr.io/clovalink/clovalink-backend:latest
    container_name: clovalink-backend
    networks:
      default:
        aliases:
          - backend
    environment:
      - DATABASE_URL=postgres://clovalink:CHANGEME_DB_PASSWORD@clovalink-db:5432/clovalink
      - REDIS_URL=redis://clovalink-redis:6379
      - JWT_SECRET=CHANGEME_JWT_SECRET
      - STORAGE_TYPE=local
      - RUST_LOG=info
      - ENVIRONMENT=production
      - CORS_DEV_MODE=false
      - CORS_ALLOWED_ORIGINS=https://files.yourdomain.com
      - CLAMAV_ENABLED=true
      - CLAMAV_HOST=clovalink-clamav
      - CLAMAV_PORT=3310
      # Optional: encrypt files at rest on local storage (openssl rand -base64 32)
      # - ENCRYPTION_KEY=CHANGEME_ENCRYPTION_KEY
      # Optional: required for scheduled auto-backups (openssl rand -base64 48)
      # - BACKUP_MASTER_KEY=CHANGEME_BACKUP_MASTER_KEY
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - /mnt/tank/configs/clovalink/uploads:/app/uploads
    depends_on:
      - clovalink-db
      - clovalink-redis
      - clovalink-clamav

  clovalink-frontend:
    image: ghcr.io/clovalink/clovalink-frontend:latest
    container_name: clovalink-frontend
    restart: unless-stopped
    ports:
      - "8080:80"
    depends_on:
      - clovalink-backend

  clovalink-db:
    image: postgres:16-alpine
    container_name: clovalink-db
    environment:
      - POSTGRES_USER=clovalink
      - POSTGRES_PASSWORD=CHANGEME_DB_PASSWORD
      - POSTGRES_DB=clovalink
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/clovalink/postgres:/var/lib/postgresql/data

  clovalink-redis:
    image: redis:7-alpine
    container_name: clovalink-redis
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/clovalink/redis:/data

  clovalink-clamav:
    image: clamav/clamav-debian:latest
    container_name: clovalink-clamav
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/clovalink/clamav:/var/lib/clamav
    healthcheck:
      test: ["CMD", "clamdscan", "--ping", "1"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 120s
```

1. Generate a secure JWT secret: `openssl rand -base64 32`
2. Replace `CHANGEME_JWT_SECRET` with your generated value
3. Generate a database password with `openssl rand -hex 24` and replace both `CHANGEME_DB_PASSWORD` values with it
4. Update `CORS_ALLOWED_ORIGINS` to your actual domain
5. Optional: uncomment `ENCRYPTION_KEY` to encrypt uploaded files at rest, and `BACKUP_MASTER_KEY` if you want scheduled auto-backups

> If you set `ENCRYPTION_KEY`, back it up somewhere safe. Losing it means losing access to every encrypted file.
{.is-warning}

> 
> ClamAV takes 2-3 minutes on first start to download virus definitions. The backend will wait for it to be ready.
{.is-info}

# 2 · Configuration

## 2.1 Default Credentials

| Role | Email | Password |
|------|-------|----------|
| SuperAdmin | superadmin@clovalink.com | password123 |
| Admin | admin@clovalink.com | password123 |
| Manager | manager@clovalink.com | password123 |
| Employee | employee@clovalink.com | password123 |
{.dense}

> 
> **Change these passwords immediately after first login!**
{.is-danger}

## 2.2 Storage Backends

ClovaLink supports multiple storage backends. Set `STORAGE_TYPE` and add the appropriate environment variables:

**Local Storage** (default):
```
STORAGE_TYPE=local
```

**S3 / Wasabi / Backblaze B2**:
```
STORAGE_TYPE=s3
S3_BUCKET=your-bucket-name
S3_REGION=us-east-1
S3_ENDPOINT=https://s3.wasabisys.com
AWS_ACCESS_KEY_ID=your-key
AWS_SECRET_ACCESS_KEY=your-secret
USE_PRESIGNED_URLS=true
```

## 2.3 Compliance Modes

ClovaLink supports enterprise compliance modes that enforce security policies:

| Mode | Enforcements |
|------|--------------|
| HIPAA | Mandatory MFA, 15-min timeout, audit logging locked, public sharing blocked |
| SOX | MFA required, file versioning mandatory, audit trails locked |
| GDPR | Consent tracking, export logging, deletion request support |
| Standard | No restrictions |
{.dense}

## 2.4 Disabling ClamAV

If you don't need virus scanning, remove the `clovalink-clamav` service and set:
```
CLAMAV_ENABLED=false
```

| Environment Variable | Description |
|---------------------|-------------|
| `DATABASE_URL` | PostgreSQL connection string |
| `REDIS_URL` | Redis connection string |
| `JWT_SECRET` | Secret for signing tokens (min 32 chars) |
| `STORAGE_TYPE` | `local` or `s3` |
| `CORS_ALLOWED_ORIGINS` | Comma-separated allowed origins |
| `CLAMAV_ENABLED` | Enable/disable virus scanning |
| `ENCRYPTION_KEY` | Optional. Base64 32-byte key for at-rest encryption of local storage |
| `BACKUP_MASTER_KEY` | Optional. Required for scheduled auto-backups |
{.dense}

# <img src="/youtube.png" class="tab-icon"> 3 · Video

*No video yet — check back soon!*