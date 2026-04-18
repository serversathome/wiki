---
title: Seafile
description: A guide to deploying Seafile
published: true
date: 2026-04-18T12:54:33.462Z
tags: 
editor: markdown
dateCreated: 2026-04-18T12:46:01.658Z
---

# <img src="/seafile.png" class="tab-icon"> What is Seafile?

**Seafile** is an open-source, self-hosted file sync and share platform focused on reliability, performance, and privacy. Written in C rather than PHP, it uses block-level syncing to transfer only the changed portions of a file, making it significantly faster and lighter than alternatives like Nextcloud or ownCloud.

Seafile organizes files into **libraries** (repositories) that can be shared, encrypted, versioned, and synced across desktop, mobile, and web clients. Version 13 introduced custom file properties, AI-generated tags, flexible views (table, gallery, kanban, map), the collaborative SeaDoc editor, and built-in wikis — pushing Seafile well beyond simple file sync into proper knowledge management territory.

The Community Edition (CE) is fully free and open source. The Professional Edition adds features like online garbage collection, S3/OpenStack storage backends, SAML SSO, audit logs, and full-text search via ElasticSearch — free for up to 3 users.


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Seafile

Before deploying, generate a JWT private key. Seafile 13 requires this for internal service authentication:

```bash
openssl rand -base64 40
```

Copy the output — you'll paste it into the compose file below as `JWT_PRIVATE_KEY`.

```yaml
services:
  db:
    image: mariadb:10.11
    container_name: seafile-db
    environment:
      - MYSQL_ROOT_PASSWORD=change_this_root_password
      - MYSQL_LOG_CONSOLE=true
      - MARIADB_AUTO_UPGRADE=1
    volumes:
      - /mnt/tank/configs/seafile/mysql:/var/lib/mysql
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 20s
      start_period: 30s
      timeout: 5s
      retries: 10

  redis:
    image: redis:7
    container_name: seafile-redis
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 20s
      timeout: 5s
      retries: 5

  seafile:
    image: seafileltd/seafile-mc:13.0-latest
    container_name: seafile
    ports:
      - "8000:80"
    volumes:
      - /mnt/tank/configs/seafile/data:/shared
    environment:
      # Core
      - SEAFILE_SERVER_HOSTNAME=seafile.example.com
      - SEAFILE_SERVER_PROTOCOL=https
      - TIME_ZONE=America/New_York
      - JWT_PRIVATE_KEY=paste_your_generated_key_here

      # Database
      - DB_HOST=db
      - DB_ROOT_PASSWD=change_this_root_password
      - SEAFILE_MYSQL_DB_HOST=db
      - SEAFILE_MYSQL_DB_USER=seafile
      - SEAFILE_MYSQL_DB_PASSWORD=change_this_db_password
      - SEAFILE_MYSQL_DB_CCNET_DB_NAME=ccnet_db
      - SEAFILE_MYSQL_DB_SEAFILE_DB_NAME=seafile_db
      - SEAFILE_MYSQL_DB_SEAHUB_DB_NAME=seahub_db

      # Cache
      - CACHE_PROVIDER=redis
      - REDIS_HOST=redis
      - REDIS_PORT=6379

      # First-run admin bootstrap (ignored on subsequent starts)
      - INIT_SEAFILE_ADMIN_EMAIL=admin@example.com
      - INIT_SEAFILE_ADMIN_PASSWORD=change_this_admin_password
      - INIT_SEAFILE_MYSQL_ROOT_PASSWORD=change_this_root_password
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped
```


> The `INIT_*` variables are **only used once** — at first startup to create the admin user and seed the database. Changing them later has no effect. Password changes after setup are done inside the Seafile web UI.
{.is-warning}

> 
> `SEAFILE_SERVER_HOSTNAME` is critical. Seafile writes this value into `seahub_settings.py` on first run, and download links, share links, and client sync URLs are all built from it. If you change your domain later, you'll need to edit `conf/seahub_settings.py` manually inside the volume — or nuke the data volume and start fresh.
{.is-danger}

# 2 · Reverse Proxy

Because Seafile uses long-lived WebSocket connections (for notifications and SeaDoc co-editing) and needs to serve both the Seahub web app and the separate `/seafhttp` file server, reverse proxy configuration is more involved than most containers.

## 2.1 Cloudflare Tunnel

For a Cloudflare Tunnel, create a public hostname pointing to `http://[SERVERIP]:8000` and in the tunnel's application settings enable:

- **HTTP/2 connection** — On
- **No TLS Verify** — On (if using self-signed upstream)
- **Disable Chunked Encoding** — Off

Cloudflare's free plan caps uploads at **100 MB per request**. For larger files, either use a paid Cloudflare plan or bypass the tunnel with a direct reverse proxy for the `/seafhttp` path.

## 2.2 Nginx Proxy Manager / Traefik

Point your proxy host to `http://[SERVERIP]:80` and make sure **WebSockets** are enabled. Seafile's internal Nginx already handles the `/seafhttp` and `/notification` routing internally, so you only need to expose the single upstream.

In NPM, under the **Advanced** tab, add:

```nginx
client_max_body_size 0;
proxy_request_buffering off;
proxy_read_timeout 310s;
```

Setting `client_max_body_size 0` disables the upload size limit entirely — essential for Seafile, which is often used to sync very large files.

# 3 · Initial Configuration

## 3.1 First Login

Log in at your Seafile URL with the `INIT_SEAFILE_ADMIN_EMAIL` and `INIT_SEAFILE_ADMIN_PASSWORD` values from the compose file. You'll land on the **My Libraries** view.

## 3.2 Create Your First Library

A **library** is Seafile's unit of storage — equivalent to a folder that you can sync, share, or encrypt independently. From the main view:

1. Click **New Library**
2. Give it a name (e.g. `Documents`, `Photos`)
3. Optionally enable **encryption** with a password (client-side, zero-knowledge — even the admin can't read it)
4. Click **Submit**

> 
> Encrypted libraries cannot be previewed in the web UI or indexed for search. If you want full-text search or online editing, leave encryption off and rely on transport (HTTPS) and at-rest (ZFS encryption) protection instead.
{.is-info}

## 3.3 Install the Desktop Client

Download Seafile Drive or Seafile Sync from [seafile.com/en/download](https://www.seafile.com/en/download/):

- **Seafile Drive Client** — Mounts libraries as a virtual drive letter; files download on-demand. Best for laptops with limited storage.
- **Seafile Sync Client** — Traditional two-way sync like Dropbox; keeps a local copy of everything. Best for desktops and primary workstations.

Add your server URL (with `https://`), email, and password. Seafile's sync engine is the reason people stay on this platform — it's fast, reliable, and handles interrupted transfers gracefully.

# 4 · Features Worth Exploring

| Feature | What it does |
|---------|--------------|
| **File Properties** | Assign custom metadata (owner, status, tags, dates) to any file and filter libraries by those fields |
| **Views** | Same library, different presentations — table, kanban, gallery, or map |
| **SeaDoc** | Collaborative real-time document editor with markdown, embeds, and version history |
| **Wikis** | Built-in hierarchical wiki with nested pages, access controls, and version tracking |
| **Share Links** | Password-protected, time-limited, permission-scoped share links for files and folders |
| **Snapshots** | Every library keeps version history automatically; restore any file to any point in time |


# 5 · Maintenance

## 5.1 Updating

Bump the image tag in your compose file (e.g. `13.0-latest` → `13.1-latest`), then redeploy. For **minor** version bumps (13.0.x → 13.0.y), this is all you need. For **major** version bumps (12 → 13), always read the [upgrade notes](https://manual.seafile.com/latest/upgrade/upgrade_notes_for_13.0.x/) first — schema migrations and config changes are sometimes required.

> 
> Before any major upgrade: stop the stack, snapshot the `/mnt/tank/configs/seafile` dataset in TrueNAS, and then proceed. ZFS snapshots make rollback trivial if the upgrade goes sideways.
{.is-success}

## 5.2 Garbage Collection

Deleted files and old block references aren't removed from disk automatically in CE — they accumulate until you run garbage collection. For the Community Edition this requires stopping Seafile briefly:

```bash
docker exec seafile /scripts/gc.sh
```

Seafile will stop, clean unreferenced blocks, and restart automatically. Run this monthly or after large deletions. Professional Edition supports online GC without downtime.



# <img src="/youtube.png" class="tab-icon"> 6 · Video

