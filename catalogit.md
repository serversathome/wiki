---
title: CatalogIT
description: A guide to deploying CatalogIT
published: true
date: 2026-05-30T10:30:25.104Z
tags: 
editor: markdown
dateCreated: 2026-05-30T10:30:25.104Z
---

# What is CatalogIT?

**CatalogIT** is a self-hosted web app for tracking the hardware and software an IT team is responsible for — an asset and license inventory with renewal reminders, attachments, and an audit trail. Think of it as a lightweight, self-hosted alternative to commercial IT asset management (ITAM) tools: register devices and software services, attach files, get notified before licenses or warranties expire, and keep a record of who changed what.


# <img src="/docker.png" class="tab-icon"> 1 · Deploy CatalogIT



```yaml
services:
  db:
    image: postgres:16
    container_name: catalogit-db
    environment:
      POSTGRES_USER: ${POSTGRES_USER:-catalogit}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB:-catalogit}
    volumes:
      - /mnt/tank/configs/catalogit/postgres:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-catalogit}"]
      interval: 5s
      timeout: 3s
      retries: 5
    restart: unless-stopped

  minio:
    image: minio/minio
    container_name: catalogit-minio
    command: server /data --console-address ":9001"
    ports:
      - "9001:9001"   # console — handy on first boot to create the bucket; can be removed later
    environment:
      MINIO_ROOT_USER: ${MINIO_ROOT_USER:-catalogit}
      MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}
    volumes:
      - /mnt/tank/configs/catalogit/minio:/data
    healthcheck:
      test: ["CMD", "mc", "ready", "local"]
      interval: 5s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  api:
    image: jonymaster/catalogit-api:${CATALOGIT_TAG:-1.4.4}
    container_name: catalogit-api
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: ${DATABASE_URL}
      JWT_SECRET: ${JWT_SECRET}
      JWT_ALGORITHM: ${JWT_ALGORITHM:-HS256}
      JWT_EXPIRY_HOURS: ${JWT_EXPIRY_HOURS:-8}
      FRONTEND_URL: ${FRONTEND_URL}
      PUBLIC_BASE_URL: ${PUBLIC_BASE_URL}
      INTEGRATION_SECRET_KEY: ${INTEGRATION_SECRET_KEY}
      SCIM_TOKEN: ${SCIM_TOKEN}
      ADMIN_DEFAULT_PASSWORD: ${ADMIN_DEFAULT_PASSWORD}
      ADMIN_EMAIL: ${ADMIN_EMAIL}
      ADMIN_EXTERNAL_ID: ${ADMIN_EXTERNAL_ID}
      ADMIN_FIRST_NAME: ${ADMIN_FIRST_NAME}
      ADMIN_LAST_NAME: ${ADMIN_LAST_NAME}
      SEED_SAMPLE_DATA: ${SEED_SAMPLE_DATA:-false}
      MINIO_ENDPOINT: ${MINIO_ENDPOINT:-http://minio:9000}
      MINIO_ACCESS_KEY: ${MINIO_ACCESS_KEY:-catalogit}
      MINIO_SECRET_KEY: ${MINIO_SECRET_KEY}
      MINIO_BUCKET_NAME: ${MINIO_BUCKET_NAME:-catalogit-attachments}
      MINIO_USE_SSL: ${MINIO_USE_SSL:-false}
      CRON_SECRET: ${CRON_SECRET}
      LOG_FORMAT: ${LOG_FORMAT:-text}
      AUDIT_RETENTION_DAYS: ${AUDIT_RETENTION_DAYS:-90}
    depends_on:
      db:
        condition: service_healthy
      minio:
        condition: service_healthy
    restart: unless-stopped

  ui:
    image: jonymaster/catalogit-ui:${CATALOGIT_TAG:-1.4.4}
    container_name: catalogit-ui
    ports:
      - "8080:80"
    depends_on:
      - api
    restart: unless-stopped

  cron:
    image: alpine/curl:latest
    container_name: catalogit-cron
    entrypoint: /bin/sh
    command:
      - -c
      - |
        echo "0 6 * * * curl -sf -X POST -H 'X-Cron-Secret: $${CRON_SECRET}' http://api:8000/api/internal/notifications/renewal-dispatch >> /proc/1/fd/1 2>&1" > /etc/crontabs/root
        crond -f -l 2
    environment:
      CRON_SECRET: ${CRON_SECRET}
    depends_on:
      - api
    restart: unless-stopped
```



## 1.1 · Environment File

```bash
# =============================================================================
# CatalogIT — environment template
# =============================================================================
# Copy to `.env` and edit. Never commit `.env`.
#
# Used by:
#   - Backend (Pydantic Settings): all keys under "Application (backend API)" below.
#   - `docker compose` (dev): substitutes `${...}`; `api` uses `env_file: .env`.
#   - `docker compose -f docker-compose.server.yml`: substitutes `${...}`; `api` passes
#     the same Application keys via `environment:` (no env_file — works with Portainer Git stacks).
#   - Services `db` / `minio`: POSTGRES_* and MINIO_ROOT_*.
# =============================================================================

# -----------------------------------------------------------------------------
# Application (backend API) — matches `app.config.Settings`
# -----------------------------------------------------------------------------

# PostgreSQL (async). User/password/db must match POSTGRES_* when using Compose `db`.
# Host `db` only resolves inside Compose; from the host use localhost + published port.
DATABASE_URL=postgresql+asyncpg://catalogit:catalogit_local@db:5432/catalogit

# HMAC secret for signing JWTs (HS256). Generate a strong value:
#   openssl rand -hex 64
JWT_SECRET=change-me-to-a-random-secret
JWT_ALGORITHM=HS256
JWT_EXPIRY_HOURS=8

# Browser origin (scheme + host + port). CORS and auth redirects.
# Local Vite: http://localhost:5173
# Production: https://catalogit.example.com
FRONTEND_URL=http://localhost:5173

# Where this API is reachable publicly (OAuth callbacks, integration metadata, links).
# Local: http://localhost:8000 — production often matches FRONTEND_URL when NPM routes /api here.
PUBLIC_BASE_URL=http://localhost:8000

# Fernet key (url-safe base64, 32 bytes) for encrypting integration tokens at rest.
# Generate: python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
INTEGRATION_SECRET_KEY=

# Bearer token for SCIM 2.0 provisioning — use a long random value in production.
# Generate:
#   openssl rand -base64 48 | tr -d '\n'
SCIM_TOKEN=change-me-to-a-random-scim-token

# First-run bootstrap admin (only when no admin user exists yet).
ADMIN_DEFAULT_PASSWORD=changeme
ADMIN_EMAIL=admin@catalogit.local
ADMIN_EXTERNAL_ID=local:admin
ADMIN_FIRST_NAME=Admin
ADMIN_LAST_NAME=User

# When true, first startup loads bundled JSON if the DB has no services yet (backend/sample_data; image: /app/sample_data). Prefer false in production.
SEED_SAMPLE_DATA=true

# S3-compatible storage (Compose service `minio` in dev). MINIO_ACCESS_KEY / MINIO_SECRET_KEY
# must match MINIO_ROOT_USER / MINIO_ROOT_PASSWORD when using Compose `minio`.
MINIO_ENDPOINT=http://minio:9000
MINIO_ACCESS_KEY=catalogit
MINIO_SECRET_KEY=catalogit_local
MINIO_BUCKET_NAME=catalogit-attachments
MINIO_USE_SSL=false

# Shared secret for internal HTTP: POST /api/internal/notifications/renewal-dispatch
# (header X-Cron-Secret), audit retention, etc. Set in production.
# Generate:
#   openssl rand -base64 48 | tr -d '\n'
CRON_SECRET=

# Logging: "text" or "json" (structured stdout; useful behind log aggregation).
LOG_FORMAT=text

# Audit rows older than this many days are eligible for purge via POST /api/internal/audit-retention.
AUDIT_RETENTION_DAYS=90

# -----------------------------------------------------------------------------
# Docker Compose: PostgreSQL (service `db`)
# -----------------------------------------------------------------------------

POSTGRES_USER=catalogit
POSTGRES_PASSWORD=catalogit_local
POSTGRES_DB=catalogit

# -----------------------------------------------------------------------------
# Docker Compose: MinIO (service `minio`)
# -----------------------------------------------------------------------------

MINIO_ROOT_USER=catalogit
MINIO_ROOT_PASSWORD=catalogit_local

# -----------------------------------------------------------------------------
# Docker Compose: server file only (`docker-compose.server.yml`)
# -----------------------------------------------------------------------------
# Not consumed by the backend directly — used for volume paths, published ports, image tags.
# Portainer: set these in the stack Environment together with all Application keys above.
#
# Required by server compose (bind mounts — create directories on the host first):
# DB_PATH=/srv/catalogit/postgres
# MINIO_DATA_PATH=/srv/catalogit/minio
#
# Optional (defaults shown in compose):
# API_PUBLISH=8000
# UI_PUBLISH=8080
# CATALOGIT_TAG=latest
# Pin a specific release on Docker Hub (must match pushed tags from `make release`):
# CATALOGIT_TAG=1.1.1

# -----------------------------------------------------------------------------
# Optional: renewals job without HTTP cron (same DATABASE_URL as the API)
# -----------------------------------------------------------------------------
# docker compose exec api python -m app.jobs.run_renewal_reminders

# -----------------------------------------------------------------------------
# Reference — every variable name (server / Portainer)
# -----------------------------------------------------------------------------
# Application: DATABASE_URL, JWT_SECRET, JWT_ALGORITHM, JWT_EXPIRY_HOURS, FRONTEND_URL,
#   PUBLIC_BASE_URL, INTEGRATION_SECRET_KEY, SCIM_TOKEN, ADMIN_DEFAULT_PASSWORD, ADMIN_EMAIL,
#   ADMIN_EXTERNAL_ID, ADMIN_FIRST_NAME, ADMIN_LAST_NAME, SEED_SAMPLE_DATA, MINIO_ENDPOINT,
#   MINIO_ACCESS_KEY, MINIO_SECRET_KEY, MINIO_BUCKET_NAME, MINIO_USE_SSL, CRON_SECRET,
#   LOG_FORMAT, AUDIT_RETENTION_DAYS
# DB / MinIO images: POSTGRES_USER, POSTGRES_PASSWORD, POSTGRES_DB, MINIO_ROOT_USER,
#   MINIO_ROOT_PASSWORD
# Server compose only: DB_PATH, MINIO_DATA_PATH, API_PUBLISH, UI_PUBLISH, CATALOGIT_TAG (api/ui image tag)
```

Key variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `DATABASE_URL` | — | Async Postgres URL, e.g. `postgresql+asyncpg://catalogit:PASS@db:5432/catalogit` (**required**) |
| `POSTGRES_PASSWORD` | — | Postgres password; must match the password in `DATABASE_URL` (**required**) |
| `JWT_SECRET` | — | HMAC secret for signing JWTs (**required**) |
| `INTEGRATION_SECRET_KEY` | — | Fernet key encrypting integration tokens at rest (**required**) |
| `SCIM_TOKEN` | — | Bearer token for SCIM provisioning (set even if unused) |
| `CRON_SECRET` | — | Shared secret for the internal renewal-dispatch endpoint |
| `FRONTEND_URL` | `http://localhost:5173` | Browser origin — set to your public URL (CORS + auth redirects) |
| `PUBLIC_BASE_URL` | `http://localhost:8000` | Where the API is reachable; usually matches `FRONTEND_URL` behind a proxy |
| `ADMIN_EMAIL` | `admin@catalogit.local` | Bootstrap admin login, created when no admin exists yet |
| `ADMIN_DEFAULT_PASSWORD` | `changeme` | Bootstrap admin password — **change this** |
| `MINIO_ROOT_PASSWORD` / `MINIO_SECRET_KEY` | — | MinIO credentials; the two must match (**required**) |
| `MINIO_BUCKET_NAME` | `catalogit-attachments` | S3 bucket for attachments |
| `SEED_SAMPLE_DATA` | `false` | Load bundled sample data on first boot (handy for a test run) |
| `AUDIT_RETENTION_DAYS` | `90` | Days before audit rows become eligible for purge |
{.dense}

> 
> Set `FRONTEND_URL` and `PUBLIC_BASE_URL` to your real public HTTPS URL **before** first login, and change `ADMIN_DEFAULT_PASSWORD` from `changeme`. The bootstrap admin is created on first startup against an empty database. Also set `SEED_SAMPLE_DATA=true` only for an initial kick-the-tyres run — leave it `false` for a real deployment.
{.is-danger}

## 1.2 · Create the MinIO Bucket

Attachments live in the MinIO bucket named by `MINIO_BUCKET_NAME` (default `catalogit-attachments`). If uploads fail on first use, the bucket likely needs creating: open the MinIO console at `http://<your-host>:9001`, log in with your `MINIO_ROOT_USER` / `MINIO_ROOT_PASSWORD`, and create a bucket with that exact name. Once it's there you can remove the `9001` port mapping if you'd rather keep the console off the network.

# 2 · Configuration

## 2.1 First Login

After the stack is healthy, the UI is at `http://<your-host>:8080` and the API docs (Swagger) at `http://<your-host>:8000/docs`. Log in with the `ADMIN_EMAIL` / `ADMIN_DEFAULT_PASSWORD` you set, then create your real users and start registering assets.

## 2.2 Reverse Proxy (single origin + path routing)

This is the part that trips people up. The frontend's API client uses a baseURL of `/`, so the **UI and API must share one hostname**, with these paths routed to the API and everything else to the UI:

| Path | Upstream |
|------|----------|
| `/` | `ui:80` |
| `/api`, `/auth`, `/scim` | `api:8000` |
| `/docs`, `/openapi.json` | `api:8000` |
{.dense}

The project documents this with Nginx Proxy Manager, but the same routing works with plain Nginx or a Cloudflare Tunnel using multiple public hostnames/paths pointed at the two services. A minimal Nginx server block:

```nginx
server {
    listen 443 ssl;
    server_name catalogit.serversatho.me;

    location ~ ^/(api|auth|scim|docs|openapi.json) {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

> 
> With this single-origin setup, `FRONTEND_URL` and `PUBLIC_BASE_URL` should both be your one public URL (e.g. `https://catalogit.serversatho.me`), since the browser keeps a single origin and the proxy splits traffic by path.
{.is-info}


