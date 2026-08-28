---
title: Bookorbit
description: A guide to deploying Bookorbit
published: true
date: 2026-08-28T20:18:42.561Z
tags: 
editor: markdown
dateCreated: 2026-08-28T20:18:42.561Z
---

# <img src="/bookorbit.png" class="tab-icon"> What is BookOrbit?

**BookOrbit** is a self-hosted library management and reading platform for ebooks, PDFs, audiobooks, and comics. It ships built-in web readers for EPUB/MOBI/AZW3, PDF, CBZ/CBR, and M4B/MP3 audiobooks, so nothing extra is needed on the client side.

Where it separates itself from the usual suspects is sync. Progress and annotations move bidirectionally between Kobo devices, KOReader, and the BookOrbit web reader, so a highlight made on an e-ink device shows up in the browser and vice versa. On top of that it pulls metadata from Google Books, Amazon, Goodreads, Open Library, Audible, and ComicVine, pushes reading activity to Hardcover and highlights to Readwise, serves OPDS, supports Send-to-Kindle, and handles multiple users with OIDC/SSO against Authentik, Keycloak, or Authelia.

It is written in TypeScript/Vue on a NestJS backend, licensed AGPL-3.0, and backed by PostgreSQL with the pgvector extension.

> BookOrbit requires a PostgreSQL database with the `uuid-ossp`, `pg_trgm`, and `vector` (pgvector) extensions. The compose file below brings up its own database, so there is nothing to install separately.
{.is-info}

# 1 · Deploy BookOrbit
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

```yaml
services:
  bookorbit:
    image: ghcr.io/bookorbit/bookorbit:latest
    container_name: bookorbit
    init: true
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
      PORT: 3000
      APP_URL: http://192.168.1.100:3000
      POSTGRES_HOST: postgres
      POSTGRES_PORT: 5432
      POSTGRES_USER: bookorbit
      POSTGRES_PASSWORD: CHANGE_ME_DB_PASSWORD
      POSTGRES_DB: bookorbit
      JWT_SECRET: CHANGE_ME_JWT_SECRET
      SETUP_BOOTSTRAP_TOKEN: CHANGE_ME_BOOTSTRAP_TOKEN
      LIBRARY_BROWSE_ROOT: /books
      NODE_MAX_OLD_SPACE_SIZE: 2048
      PUID: 1000
      PGID: 1000
    volumes:
      - /mnt/tank/media:/media
      - /mnt/tank/configs/bookorbit:/data
    depends_on:
      postgres:
        condition: service_healthy

  postgres:
    image: pgvector/pgvector:pg18
    container_name: bookorbit-db
    restart: unless-stopped
    environment:
      POSTGRES_USER: bookorbit
      POSTGRES_PASSWORD: CHANGE_ME_DB_PASSWORD
      POSTGRES_DB: bookorbit
      PGDATA: /var/lib/postgresql/data/pgdata
    volumes:
      - /mnt/tank/configs/bookorbit-db:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U bookorbit -d bookorbit"]
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 20s
```

1. Generate the three secrets before deploying:

```bash
openssl rand -hex 24   # POSTGRES_PASSWORD
openssl rand -hex 32   # JWT_SECRET
openssl rand -hex 16   # SETUP_BOOTSTRAP_TOKEN
```

2. Set `POSTGRES_PASSWORD` to the same value in **both** services or the app will not connect.
3. Set `APP_URL` to the address you will actually browse to. It is used for Kobo sync endpoints and outbound email, so a wrong value here breaks device sync later.


> `APP_URL` is baked into the Kobo sync URLs handed out to devices. If you plan to put BookOrbit behind a reverse proxy, set it to the final public hostname now rather than changing it after your devices are paired.
{.is-warning}

## <img src="/truenas.png" class="tab-icon"> TrueNAS

BookOrbit is in the **community** train, added June 2026 and tracking upstream closely.

1. Navigate to **Apps** in the TrueNAS UI and select **Discover Apps**
2. Search for **BookOrbit** and click **Install**
3. Under **BookOrbit Configuration**, set:
   - **Timezone**: your local zone
   - **Postgres Image**: leave at **Postgres 18**
   - **Database Password**, **JWT Secret**, **Bootstrap Token**: generate each with `openssl rand -hex 32`
4. Under **Network Configuration**, note the **WebUI Port** default of `30441`
5. Under **Storage Configuration**, switch these from ixVolume to **Host Path** where you want control:
   - **Books Storage**: `/mnt/tank/media/books`
   - **Data Storage**: `/mnt/tank/configs/bookorbit`
   - **Book Drop Storage**: `/mnt/tank/media/books/dock`
   - **Postgres Data Storage**: leave as an ixVolume unless you have a reason not to
6. Under **Resources Configuration**, the defaults are 2 CPUs and 4096 MB memory. Raise the memory for very large libraries.
7. Click **Install**

> Do not change the Postgres image version after the database has been initialized. It triggers a one-way upgrade, and picking an older version afterward will refuse to start. Snapshot the dataset first.
{.is-danger}

> If you use a Host Path for Postgres data, tick **Automatic Permissions** so TrueNAS chowns the directory for the postgres container.
{.is-info}

# 2 · First-Time Setup

## 2.1 Setup Wizard

On first load BookOrbit presents a setup screen that asks for the bootstrap token.

1. Paste the `SETUP_BOOTSTRAP_TOKEN` value you generated
2. Create the admin account: email, username, password
3. Finish the wizard and sign in

> The bootstrap token is single-use for the setup endpoint. Once the admin account exists, it stops being a way in, but there is no reason to leave it in a shared compose file either.
{.is-success}

## 2.2 Creating a Library

1. Go to **Settings > Libraries** and click **Add Library**
2. Give it a name and pick the folder inside `/books`
3. Choose the content type — Books, Audiobooks, or Comics — since it drives which reader and which metadata providers get used
4. Set format priority if the same title exists as both EPUB and PDF
5. Save, then run a scan

> **Folder Layout That Scans Cleanly**
> One folder per author, subfolder per title
> Keep audiobook parts together in a single title folder
> Separate libraries for comics and books rather than one mixed library
{.is-success}


## 2.3 Metadata Providers

Under **Settings > Metadata**, enable and order the providers you want. Google Books and Open Library cover general fiction and non-fiction well, Audible is the one that matters for audiobooks, and ComicVine is effectively required for comics. Order matters — the first provider that returns a match wins unless you override the field manually.

## 2.4 Book Dock

Set `BOOK_DOCK_PATH` (or the Book Drop storage on TrueNAS) to a watched folder and anything dropped there gets imported automatically. Useful as an SMB share target so you can copy a file from a laptop and have it appear in the library without touching the UI.

# 3 · Device Sync

## 3.1 KOReader Plugin

The plugin adds progress sync, two-way annotation sync, and a catalog browser that lets you search and download from the library on the device itself.

1. In BookOrbit go to **Settings > Integrations > KOReader** and click **Download Plugin**
2. Unzip `bookorbit.koplugin.zip`
3. Copy the `bookorbit.koplugin` folder to `koreader/plugins/` on the device
4. Restart KOReader and open a book
5. Use **Tools > BookOrbit Sync** to connect

> The downloaded plugin is pre-configured with your server URL and credentials, so there is no typing a long URL on an e-ink keyboard.
{.is-success}

## 3.2 Kobo Sync

Kobo devices sync natively against BookOrbit's endpoints, which is why `APP_URL` has to be correct and reachable from wherever the device is. Highlights and deletes propagate in both directions and merge with web reader annotations into the same hub.

## 3.3 Hardcover and Readwise

- **Hardcover**: pushes status, progress, reading dates, and ratings on configurable triggers. It can also pull your read history back to backfill entries that are blank in BookOrbit.
- **Readwise**: pushes new highlights and notes as they are created, from both the web reader and synced devices, matching covers by ISBN.

Both are configured under **Settings > Integrations** with an API token from the respective service.

# 4 · Delivery and Access

| Method | Where to configure | Notes |
|--------|--------------------|-------|
| OPDS | Settings > Integrations | Works with any OPDS-capable reader app |
| Send-to-Kindle | Settings > Email | Requires SMTP credentials |
| Drag and drop | Library view | Browser upload for one-off files |
| Book Dock | Settings > Libraries | Watched folder, hands-free import |


> If you store SMTP credentials, set `EMAIL_ENCRYPTION_KEY` to a value from `openssl rand -hex 32`. Without it those credentials sit unencrypted in the database.
{.is-warning}

# 5 · Reverse Proxy

BookOrbit ships a strict CSP and expects to be served from the hostname in `APP_URL`. A minimal Nginx Proxy Manager setup:

1. Create a proxy host pointing at `bookorbit:3000` (or the host IP and port)
2. Enable **Websockets Support**
3. Request a Let's Encrypt certificate and enable **Force SSL**
4. Update `APP_URL` to the HTTPS hostname and recreate the container
5. Re-pair Kobo devices if they were paired against the old URL

> If the frontend is served from a different domain than `APP_URL`, set `CLIENT_URL` to that domain or CORS will block it.
{.is-info}

# 6 · Multi-User and SSO

Under **Settings > Users** you get granular per-user permissions with isolated reading data — each user has their own progress, highlights, statistics, and goals against the shared library.

For OIDC, BookOrbit has native support for Authentik, Keycloak, and Authelia. Configure the issuer URL, client ID, and secret under **Settings > Authentication**.

> If your identity provider lives on a private address, set `OIDC_ALLOW_LOCAL_ISSUERS=true`. Leave it off on anything internet-facing.
{.is-warning}


