---
title: Sharely
description: A guide to deploying Sharely
published: true
date: 2026-05-30T10:09:48.539Z
tags: 
editor: markdown
dateCreated: 2026-05-30T10:09:48.539Z
---

# <img src="/sharely.png" class="tab-icon"> What is Sharely?

**Sharely** is a self-hosted file sharing platform with a clean web interface, ShareX integration, and full API access. Upload screenshots, files, and media, then instantly share them via short 8-character links. It's a modern, actively-developed alternative to XBackBone (and includes a built-in importer for migrating away from it).

Notable features:

- **Drag-and-drop web UI** with a searchable gallery, type filters (images, video, audio, PDF, code), and full mobile responsiveness
- **Chunked uploads** for files up to 2 GB, uploaded in parallel multi-part chunks with no client config
- **ShareX integration** via a one-click `.sxcu` config download for automatic Windows screenshot uploads
- **API uploads** with Bearer token auth (curl, wget, any HTTP client)
- **Inline viewing** — images zoom, audio/video stream with seek (HTTP Range), PDFs render in-browser, code is syntax-highlighted
- **Auto thumbnails** for video and PDF files (ffmpeg + ghostscript bundled in the Docker image)
- **Embed modes** — per-user toggle between rich Open Graph / Twitter Card embeds and raw direct-file redirects, with automatic social bot detection
- **Real-time UI** over WebSockets — live view counters, gallery refresh, admin stats, and audit log streaming
- **Admin dashboard** with stats, user/file management, and a CSV-exportable audit log
- **8 languages**, email verification / password reset (SMTP), role-based access control, and GDPR-ready privacy tooling


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Sharely

Since there's no published container image, clone the repo into your configs directory first so Dockge/Compose can build it locally:

```bash
cd /mnt/tank/configs
git clone https://github.com/Christianoooooo/sharely.git
cd sharely
cp .env.example .env
```

The official `docker-compose.yml` is ready to use as-is and reads its settings from a `.env` file. The compose below is lightly adapted to use **bind mounts** under `/mnt/tank/configs/sharely` instead of named volumes, so your uploads and database live on your `tank` pool.

```yaml
services:
  app:
    build: .
    container_name: sharely
    ports:
      - "3000:3000"
    environment:
      MONGODB_URI: "mongodb://${MONGO_APP_USER:-appuser}:${MONGO_APP_PASSWORD}@mongo:27017/${MONGO_DB_NAME:-sharely}?authSource=${MONGO_DB_NAME:-sharely}"
      SESSION_SECRET: "${SESSION_SECRET}"
      BASE_URL: "${BASE_URL:-http://localhost:3000}"
      SITE_NAME: "${SITE_NAME:-sharely}"
      MAX_FILE_SIZE_MB: "${MAX_FILE_SIZE_MB:-100}"
      ALLOW_REGISTRATION: "${ALLOW_REGISTRATION:-true}"
      SMTP_HOST: "${SMTP_HOST:-}"
      SMTP_PORT: "${SMTP_PORT:-587}"
      SMTP_SECURE: "${SMTP_SECURE:-false}"
      SMTP_USER: "${SMTP_USER:-}"
      SMTP_PASS: "${SMTP_PASS:-}"
      SMTP_FROM: "${SMTP_FROM:-}"
    volumes:
      - /mnt/tank/configs/sharely/uploads:/app/uploads
    depends_on:
      mongo:
        condition: service_healthy
    restart: unless-stopped

  mongo:
    image: mongo:7
    container_name: sharely-mongo
    ports:
      - "127.0.0.1:27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: "${MONGO_ROOT_PASSWORD}"
      MONGO_INITDB_DATABASE: "${MONGO_DB_NAME:-sharely}"
      MONGO_APP_USER: "${MONGO_APP_USER:-appuser}"
      MONGO_APP_PASSWORD: "${MONGO_APP_PASSWORD}"
      MONGO_DB_NAME: "${MONGO_DB_NAME:-sharely}"
    volumes:
      - /mnt/tank/configs/sharely/mongo:/data/db
      - ./scripts/mongo-init.js:/docker-entrypoint-initdb.d/mongo-init.js:ro
    restart: unless-stopped
```



The app will be available at `http://<your-host>:3000`. **The first account you register automatically becomes the admin.**

## 1.1 · Environment File



| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `3000` | Port the server listens on |
| `MONGO_ROOT_PASSWORD` | — | MongoDB root password (**required**) |
| `MONGO_APP_USER` | `appuser` | MongoDB application user |
| `MONGO_APP_PASSWORD` | — | MongoDB app user password (**required**) |
| `MONGO_DB_NAME` | `sharely` | Database name |
| `SESSION_SECRET` | — | Session encryption secret (**required**) |
| `BASE_URL` | `http://localhost:3000` | Public base URL for share links, no trailing slash |
| `SITE_NAME` | `sharely` | Name shown in Open Graph embeds |
| `MAX_FILE_SIZE_MB` | `100` | Max upload size in MB (chunked uploads still allow up to 2 GB) |
| `ALLOW_REGISTRATION` | `true` | Set `false` for admin-only user creation |
| `SMTP_HOST` | — | SMTP hostname; leave blank to disable email features |
| `SMTP_PORT` | `587` | SMTP port |
| `SMTP_SECURE` | `false` | `true` for implicit TLS (465), `false` for STARTTLS (587) |
| `SMTP_USER` | — | SMTP username |
| `SMTP_PASS` | — | SMTP password |
| `SMTP_FROM` | *(SMTP_USER)* | From address on outgoing email |


Generate a strong session secret:

```bash
openssl rand -hex 32
```

> 
> Set `BASE_URL` to your real public domain **before** uploading anything. It's baked into every generated share link, so changing it later means existing links point to the wrong host.
{.is-danger}

# 2 · Configuration

## 2.1 First Login & API Key

1. Browse to your instance and register — this first account becomes the **admin**
2. Go to **Settings → API Key** to grab your Bearer token for API/ShareX uploads
3. If you don't want open sign-ups, set `ALLOW_REGISTRATION=false` and create users from **Admin → Users**

## 2.2 ShareX Setup

1. Log in and open the **Upload** page
2. Click **Download ShareX Config**
3. Open the downloaded `.sxcu` file — ShareX imports it automatically
4. Screenshots and uploads now go straight to your instance

ShareX receives a `deletionUrl` in each upload response, so it can delete files later without re-authenticating.

## 2.3 API Usage

Upload a file:

```bash
curl -X POST https://files.serversatho.me/api/upload \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -F "sharex=@/path/to/file.png"
```

Delete a file:

```bash
curl -X DELETE https://files.serversatho.me/api/delete/SHORT_ID \
  -H "Authorization: Bearer YOUR_API_KEY"
```

List your files:

```bash
curl https://files.serversatho.me/api/gallery \
  -H "Authorization: Bearer YOUR_API_KEY"
```


# 3 · Maintenance

Because Sharely builds from source, updating means pulling the latest code and rebuilding rather than just pulling a new image:

```bash
cd /mnt/tank/configs/sharely
git pull
docker compose up -d --build
```

