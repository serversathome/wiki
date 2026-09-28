---
title: Attic
description: A guide to deploying Attic
published: true
date: 2026-09-28T18:02:18.858Z
tags: 
editor: markdown
dateCreated: 2026-09-28T18:01:07.684Z
---

# <img src="/attic-assets.png" class="tab-icon"> What is Attic?

**Attic** is a self-hosted home inventory for everything you own. Catalog appliances, tools, electronics, books, games and furniture, and keep each item's location, condition, purchase price, warranty, photos, receipts and manuals in one place. Locations nest the way your house does (room, shelf, box), categories carry their own custom fields, and imports from Google Books, TMDB, BoardGameGeek and IGDB fill in metadata and cover art for you.

<div class="glance">
  <div><span>Port</span><b><code>8095</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>2 services</b></div>
  <div><span>Depends on</span><b>Postgres</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/lmmendes/attic"><i class="mdi mdi-github"></i>Project</a>
    <a href="https://getattic.dev"><i class="mdi mdi-book-open-variant"></i>Docs</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Attic

Create the folders first:

```bash
mkdir -p /mnt/tank/configs/attic/{uploads,postgres}
chown -R 568:568 /mnt/tank/configs/attic
```

Generate a session secret and a database password:

```bash
openssl rand -base64 48
openssl rand -hex 16
```

```yaml
services:
  attic:
    image: ghcr.io/lmmendes/attic:latest
    container_name: attic
    user: "568:568"
    environment:
      - ATTIC_PORT=8080
      - ATTIC_DATABASE_URL=postgres://attic:CHANGEME_DB_PASSWORD@attic-db:5432/attic?sslmode=disable
      - ATTIC_LOCAL_STORAGE_PATH=/data/uploads
      - ATTIC_PUID=568
      - ATTIC_PGID=568
      - ATTIC_BASE_URL=http://your-server-ip:8095
      - ATTIC_CORS_ORIGINS=http://your-server-ip:8095
      - ATTIC_SESSION_SECRET=CHANGEME_SESSION_SECRET
      - ATTIC_ADMIN_EMAIL=you@example.com
      - ATTIC_ADMIN_PASSWORD=CHANGEME
      # Optional import sources
      - ATTIC_GOOGLE_BOOKS_API_KEY=
      - ATTIC_TMDB_API_KEY=
      - ATTIC_BGG_API_KEY=
      - ATTIC_IGDB_CLIENT_ID=
      - ATTIC_IGDB_CLIENT_SECRET=
    ports:
      - "8095:8080"
    volumes:
      - /mnt/tank/configs/attic/uploads:/data/uploads
    depends_on:
      attic-db:
        condition: service_healthy
    restart: unless-stopped

  attic-db:
    image: postgres:16-alpine
    container_name: attic-db
    user: "568:568"
    environment:
      - POSTGRES_USER=attic
      - POSTGRES_PASSWORD=CHANGEME_DB_PASSWORD
      - POSTGRES_DB=attic
    volumes:
      - /mnt/tank/configs/attic/postgres:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U attic"]
      interval: 5s
      timeout: 5s
      retries: 5
    restart: unless-stopped
```

1. Replace the `CHANGEME` values. The database password appears twice and must match
2. Set `ATTIC_ADMIN_EMAIL` and `ATTIC_ADMIN_PASSWORD`. These create the first admin account on first boot
3. Replace `your-server-ip` with your server's IP, or with your public URL if you put Attic behind a Cloudflare Tunnel
4. Deploy the stack. Database migrations run automatically on startup

> The admin credentials are only used when the database has no users yet. Changing them in the compose file after the first boot does nothing. If you leave them unset, Attic falls back to `admin` / `admin`, so set them before the first deploy.
{.is-info}

> The upstream compose also ships a Keycloak service for SSO. It is optional and left out here. Attic works fine with local password logins, and you can point it at any OIDC provider later with the `ATTIC_OIDC_*` variables.
{.is-info}

# 2 · Configuration

## 2.1 Initial Setup

1. Browse to `http://your-server-ip:8095` and log in with the admin email and password from the compose file
2. Build out **Locations** first (rooms, then shelves or boxes inside them) so new items have somewhere to live
3. Create **Categories** for the kinds of things you own. Custom fields added to a parent category are inherited by its children
4. Add items, attach photos, receipts and manuals, and fill in purchase dates and warranty info so the dashboard can flag expiring warranties

## 2.2 Import Sources

Attic can pull metadata and cover art for books, movies, TV, board games and video games. Each source is optional:

| Source | Variable | Notes |
|--------|----------|-------|
| Google Books | `ATTIC_GOOGLE_BOOKS_API_KEY` | Works without a key, but a key avoids Google's low shared quota |
| TMDB | `ATTIC_TMDB_API_KEY` | Movies and TV |
| BoardGameGeek | `ATTIC_BGG_API_KEY` | Board games |
| IGDB | `ATTIC_IGDB_CLIENT_ID` and `ATTIC_IGDB_CLIENT_SECRET` | Both come from a Twitch developer application |
{.dense}

## 2.3 Collections and Saved Filters

Collections group items without moving them, so a PS5 game can live on the TV stand and still belong to a "PS5 Games" collection. The asset list also supports advanced AND/OR filters across custom fields. Save the ones you use often, and pin up to five to the sidebar.

## 2.4 S3 Storage (Optional)

Attachments are stored on disk in `/mnt/tank/configs/attic/uploads` by default. To use S3-compatible storage instead, set `ATTIC_S3_ENDPOINT`, `ATTIC_S3_BUCKET`, `ATTIC_S3_REGION`, `ATTIC_S3_ACCESS_KEY` and `ATTIC_S3_SECRET_KEY`. Attic switches to S3 as soon as both keys are present.

