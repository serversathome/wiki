---
title: Neohabit
description: A guide to deploying Neohabit
published: true
date: 2026-03-02T21:01:26.673Z
tags: 
editor: markdown
dateCreated: 2026-03-02T21:01:26.673Z
---

# <img src="/neohabit.png" class="tab-icon"> What is Neohabit?

**Neohabit** is a self-hosted habit tracker designed for systematic self-improvement. Unlike basic trackers, Neohabit supports flexible habits that happen X times in Y days, custom heatmaps (monochromatic, numeric, and fractured styles), skill trees for planning progressions, and project grouping to organize related habits together. It features a polished desktop-focused interface with a responsive mobile browser view, multiple account support, and dark/light themes.


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Neohabit

Neohabit uses a multi-container stack with a Go backend, React frontend, PostgreSQL database, and Caddy reverse proxy.

First, create a directory for the configuration files and download them:

```bash
mkdir -p /mnt/tank/stacks/neohabit && cd /mnt/tank/stacks/neohabit
```

```bash
wget -O docker-compose.yaml https://raw.githubusercontent.com/Vsein/Neohabit/refs/heads/main/docker-compose.yaml &&
wget -O .env https://raw.githubusercontent.com/Vsein/Neohabit/refs/heads/main/.env.example &&
wget -O Caddyfile https://raw.githubusercontent.com/Vsein/Neohabit/refs/heads/main/Caddyfile
```

Next, edit the `.env` file and set the **required** values:

```bash
nano .env
```

At minimum, set these two values:

| Variable | Description |
|----------|-------------|
| `POSTGRES_PASSWORD` | Database password (required) |
| `JWT_SECRET` | Authentication secret, use 32+ random bytes (required) |
{.dense}

> 
> You can generate a strong JWT secret with: `openssl rand -base64 32`
{.is-info}

Here is the full compose file for reference:

```yaml
services:
  postgres:
    restart: unless-stopped
    image: postgres:18-alpine
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-postgres} -d ${POSTGRES_DB:-neohabit}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 5s
      start_interval: 2s
    environment:
      POSTGRES_DB: ${POSTGRES_DB:-neohabit}
      POSTGRES_USER: ${POSTGRES_USER:-postgres}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD?Missing POSTGRES_PASSWORD}
    volumes:
      - /mnt/tank/configs/neohabit/postgres_data:/var/lib/postgresql

  backend:
    restart: unless-stopped
    image: ghcr.io/vsein/neohabit-back:${TAG:-latest}
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      PG_DSN: postgres://${POSTGRES_USER:-postgres}:${POSTGRES_PASSWORD?Missing POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB:-neohabit}?sslmode=disable
      PG_MAX_OPEN_CONNS: ${PG_MAX_OPEN_CONNS:-20}
      PG_MAX_LIFETIME: ${PG_MAX_LIFETIME:-30m}
      PG_MAX_IDLE_TIME: ${PG_MAX_IDLE_TIME:-10m}
      LOG_LEVEL: ${LOG_LEVEL:-info}
      JWT_SECRET: ${JWT_SECRET?Missing JWT_SECRET}
      FRONTEND_URL: ${FRONTEND_URL:-http://127.0.0.1:${FRONTEND_PORT:-8080}}


  frontend:
    restart: unless-stopped
    image: ghcr.io/vsein/neohabit-front:${TAG:-latest}
    environment:
      DISABLE_SIGNUP: ${DISABLE_SIGNUP:-false}
      STRICT_USER_FIELDS: ${STRICT_USER_FIELDS:-false}
      REQUIRE_EMAIL: ${REQUIRE_EMAIL:-false}
      DEMO: ${DEMO:-false}
    depends_on:
      - backend
    volumes:
      - /mnt/tank/configs/neohabit/frontend-dist:/srv


  caddy:
    restart: unless-stopped
    image: caddy:2-alpine
    ports:
      - "127.0.0.1:${FRONTEND_PORT:-8080}:80"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - /mnt/tank/configs/neohabit/frontend-dist:/srv
      - /mnt/tank/configs/neohabit/caddy_data:/data
      - /mnt/tank/configs/neohabit/caddy_config:/config
    depends_on:
      - frontend




```


# 2 · Configuration


Neohabit ships with a Caddy reverse proxy. The `Caddyfile` supports three modes:

**Localhost / LAN (default)**
The default configuration listens on `:80` and is bound to `127.0.0.1:8080`. For LAN access, uncomment the LAN port line in `docker-compose.yaml`:

```yaml
# - "127.0.0.1:${FRONTEND_PORT:-8080}:80"   # ← localhost-only (default)
- "${FRONTEND_PORT:-8080}:80"                 # ← for LAN
```

**LAN + HTTPS**
Uncomment `tls internal` and the frontend reverse proxy lines in the `Caddyfile`, then switch ports to `443` in `docker-compose.yaml`.

**Web hosting (public domain)**
Replace `:80` in the `Caddyfile` with your domain name (e.g. `neohabit.example.com`), uncomment the `tls` line with your email for automatic Let's Encrypt certificates, and expose ports `80`, `443`, and `443/udp` in the compose file.

> 
> Remember to update `FRONTEND_URL` in `.env` to match your actual URL — this is required for CORS to work properly.
{.is-danger}

