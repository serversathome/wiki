---
title: LibreDB Studio
description: A guide to deploying LibreDB Studio
published: true
date: 2026-09-15T12:35:08.688Z
tags: 
editor: markdown
dateCreated: 2026-09-15T12:32:23.123Z
---

# <img src="/libredbstudio.png" class="tab-icon"> What is LibreDB Studio?

**LibreDB Studio** is a browser-based SQL client you run on your server rather than your desktop. One container reaches sixteen database engines — PostgreSQL, MySQL, SQLite, MongoDB, Redis, ClickHouse, Elasticsearch and Cassandra among them — through the same explorer, editor and result grid.

<div class="glance">
  <div><span>Port</span><b><code>3000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/libredb/libredb-studio"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy LibreDB Studio

```yaml
services:
  libredbstudio:
    image: ghcr.io/libredb/libredb-studio:latest
    container_name: libredbstudio
    user: "568:568"
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - ADMIN_EMAIL=admin@libredb.org
      - STORAGE_PROVIDER=sqlite
      - STORAGE_SQLITE_PATH=/app/data/libredb-storage.db
    volumes:
      - /mnt/tank/configs/libredbstudio:/app/data
```

> 
> Port 3000 is busy on a lot of homelabs. If the container will not start and the error is `address already in use`, change the left-hand side only — `3001:3000`.
{.is-warning}

> 
> On first run the admin password is generated and printed to the container log. It is also written to `auth-bootstrap.json` inside the volume, so if you miss it, read it from there. To choose your own instead, add `ADMIN_PASSWORD` and a `JWT_SECRET` of at least 32 characters before the first start — a shorter secret makes the server exit rather than start.
{.is-info}

Open `http://your-server-ip:3000` and sign in as `admin@libredb.org`. You land on the admin dashboard; the query editor is behind the **Editor** button, top right.

Idle footprint is about 60 MB of RAM. The image is 292 MB and is published for `amd64` and `arm64`, so it runs on a Pi 4 or 5 as well as an x86 box.

# 2 · Configuration

## 2.1 Adding a Database

Connections are added from the sidebar. If your database runs in Docker on the same host, put both containers on the same network and use the container name as the host — `postgres`, not `localhost`:

```yaml
services:
  libredbstudio:
    networks:
      - dbnet
  postgres:
    networks:
      - dbnet

networks:
  dbnet:
    driver: bridge
```

If your database runs elsewhere, use its IP and make sure the port is reachable from this container.

For anything you only read from, connect with an account that holds `SELECT` and nothing else. The engine then refuses a stray `DELETE` with `permission denied for table ...`.

## 2.2 Storage

`STORAGE_PROVIDER=sqlite` keeps your saved connections in a file inside the volume, so they survive a container recreate. Leave it out and the default is `local`, which keeps them in the **browser** — a different browser, or a cleared cache, and they are gone.

For a multi-user setup use `STORAGE_PROVIDER=postgres` with `STORAGE_POSTGRES_URL` instead.

## 2.3 Backups and Updates

The whole state is the volume. Back up `/mnt/tank/configs/libredbstudio` and you have your connections, your saved queries and the generated credentials. Note that `auth-bootstrap.json` holds the admin password and the JWT secret in plain text, so treat that directory as a secret.

Updating is the usual pull and recreate:

```bash
docker compose pull && docker compose up -d
```

## 2.4 Optional AI Assistant

Off unless you configure it, and configured by environment variable rather than in the UI:

```yaml
environment:
  - LLM_PROVIDER=ollama
  - LLM_MODEL=qwen2.5:7b
  - LLM_API_URL=http://172.17.0.1:11434/v1
```

Three things that trip people up:

1. Ollama listens on loopback by default, so the container cannot reach it. Start it with `OLLAMA_HOST=0.0.0.0`.
2. `172.17.0.1` is the Docker bridge on Linux. On Docker Desktop use `http://host.docker.internal:11434/v1`.
3. The model has to support tool calling. `qwen2.5:7b` is 4.7 GB and the fastest of the tested set; `qwen3.5:4b` at 3.4 GB is the smallest that passes. One that cannot is refused with an explanation rather than failing silently.

> 
> The assistant reads and nothing else: 200 rows and 10 seconds per statement, enforced by the database in a read-only session rather than by inspecting the SQL. It runs on PostgreSQL, SQLite and DuckDB; on other engines it only drafts. Leave the `LLM_` variables unset and no AI appears anywhere in the interface.
{.is-info}

For Gemini or OpenAI instead, set `LLM_PROVIDER` accordingly and add `LLM_API_KEY`.

## 2.5 Reverse Proxy

Nothing unusual. Point your proxy at port 3000 on the container. In Nginx Proxy Manager that is a plain proxy host with **Websockets Support** switched on.

## 2.6 Environment Variables

| Environment Variable | Description |
|---------------------|-------------|
| `ADMIN_EMAIL` | Admin login (default `admin@libredb.org`) |
| `ADMIN_PASSWORD` | Optional. Generated on first run and written to `auth-bootstrap.json` |
| `JWT_SECRET` | Optional. Generated on first run. If you set it, 32 characters minimum |
| `USER_EMAIL` / `USER_PASSWORD` | Optional second, non-admin account |
| `STORAGE_PROVIDER` | `sqlite`, `postgres`, or `local` (browser only) |
| `STORAGE_SQLITE_PATH` | SQLite file path, e.g. `/app/data/libredb-storage.db` |
| `STORAGE_POSTGRES_URL` | Connection string when `STORAGE_PROVIDER=postgres` |
| `LLM_PROVIDER` | `ollama`, `gemini`, `openai`. Unset means no AI |
| `LLM_MODEL` | Model name, e.g. `qwen2.5:7b` |
| `LLM_API_URL` | Endpoint, e.g. `http://172.17.0.1:11434/v1` |
| `LLM_API_KEY` | Required for `gemini` and `openai` |

