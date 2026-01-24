---
title: Rybbit
description: A guide to deploying Rybbit
published: true
date: 2026-01-24T20:00:56.833Z
tags: 
editor: markdown
dateCreated: 2026-01-24T19:52:22.810Z
---

# <img src="/rybbit.png" class="tab-icon"> What is Rybbit?

**Rybbit** is an open source, privacy-first web analytics platform. It's a self-hosted alternative to Google Analytics that respects user privacy while providing detailed insights about your website traffic.

# <img src="/cloudflare.png" class="tab-icon"> Cloudflare Tunnel Setup

This guide is for deploying Rybbit behind a **Cloudflare Tunnel** reverse proxy. Rybbit requires path-based routing (`/api` goes to the backend, everything else goes to the frontend), which the Cloudflare Zero Trust dashboard doesn't support natively. To work around this, we use an nginx container to handle the routing.

## Network Requirements

This setup assumes you have a Cloudflare Tunnel container already running. You'll need a shared Docker network that both your tunnel container and Rybbit can communicate on.

### Create the Network

If you don't already have a shared network, create one:

```bash
docker network create cftunnel
```

### Connect Your Tunnel

Make sure your Cloudflare Tunnel container is on this network by adding it to your tunnel's compose file:

```yaml
services:
  cloudflared:
    image: cloudflare/cloudflared:latest
    # ... your existing config
    networks:
      - cftunnel

networks:
  cftunnel:
    name: cftunnel
    external: true
```

Then redeploy your tunnel stack.

> Replace `cftunnel` with whatever network name you prefer. Just make sure to use the same name in the Rybbit compose file below.
{.is-info}

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Rybbit

## 1.1 Create the Nginx Config

Before deploying, create the nginx configuration file at `/mnt/tank/configs/rybbit/nginx.conf`:

```nginx
events {
    worker_connections 1024;
}

http {
    server {
        listen 80;

        location /api/ {
            proxy_pass http://rybbit_backend:3001;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location / {
            proxy_pass http://rybbit_client:3002;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

## 1.2 Deploy the Stack

```yaml
services:
  rybbit_clickhouse:
    image: clickhouse/clickhouse-server:25.4.2
    container_name: rybbit_clickhouse
    volumes:
      - /mnt/tank/configs/rybbit/clickhouse-data:/var/lib/clickhouse
    environment:
      - CLICKHOUSE_DB=analytics
      - CLICKHOUSE_USER=default
      - CLICKHOUSE_PASSWORD=changeme
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:8123/ping"]
      interval: 3s
      timeout: 5s
      retries: 5
      start_period: 10s
    restart: unless-stopped
    networks:
      - internal

  rybbit_postgres:
    image: postgres:17.4
    container_name: rybbit_postgres
    environment:
      - POSTGRES_USER=rybbit
      - POSTGRES_PASSWORD=changeme
      - POSTGRES_DB=analytics
    volumes:
      - /mnt/tank/configs/rybbit/postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U rybbit -d analytics"]
      interval: 3s
      timeout: 5s
      retries: 5
      start_period: 10s
    restart: unless-stopped
    networks:
      - internal

  rybbit_backend:
    image: ghcr.io/rybbit-io/rybbit-backend:latest
    container_name: rybbit_backend
    environment:
      - NODE_ENV=production
      - CLICKHOUSE_HOST=http://rybbit_clickhouse:8123
      - CLICKHOUSE_DB=analytics
      - CLICKHOUSE_PASSWORD=changeme
      - POSTGRES_HOST=rybbit_postgres
      - POSTGRES_PORT=5432
      - POSTGRES_DB=analytics
      - POSTGRES_USER=rybbit
      - POSTGRES_PASSWORD=changeme
      - BETTER_AUTH_SECRET=generate_with_openssl_rand_hex_32
      - BASE_URL=https://rybbit.example.com
      - DISABLE_SIGNUP=false
      - MAPBOX_TOKEN=optional_for_globe_visualization
    depends_on:
      rybbit_clickhouse:
        condition: service_healthy
      rybbit_postgres:
        condition: service_started
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://127.0.0.1:3001/api/health"]
      interval: 3s
      timeout: 5s
      retries: 5
      start_period: 10s
    restart: unless-stopped
    networks:
      - internal

  rybbit_client:
    image: ghcr.io/rybbit-io/rybbit-client:latest
    container_name: rybbit_client
    environment:
      - NODE_ENV=production
      - NEXT_PUBLIC_BACKEND_URL=https://rybbit.example.com
      - NEXT_PUBLIC_DISABLE_SIGNUP=false
    depends_on:
      - rybbit_backend
    restart: unless-stopped
    networks:
      - internal

  rybbit_nginx:
    image: nginx:alpine
    container_name: rybbit_nginx
    volumes:
      - /mnt/tank/configs/rybbit/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - rybbit_client
      - rybbit_backend
    restart: unless-stopped
    networks:
      - internal
      - cftunnel

networks:
  internal:
    driver: bridge
  cftunnel:
    name: cftunnel
    external: true
```

> Replace all `changeme` passwords with secure values. Make sure `CLICKHOUSE_PASSWORD` in the backend matches the one in the clickhouse service, and `POSTGRES_USER`/`POSTGRES_PASSWORD` match between postgres and backend.
{.is-warning}

> If you named your tunnel network something other than `cftunnel`, update the network name at the bottom of the compose file.
{.is-info}

## 1.3 Configuration

| Variable | Description |
|----------|-------------|
| `BETTER_AUTH_SECRET` | Generate with `openssl rand -hex 32` |
| `BASE_URL` | Your full domain with https (e.g., `https://rybbit.example.com`) |
| `NEXT_PUBLIC_BACKEND_URL` | Same as `BASE_URL` |
| `MAPBOX_TOKEN` | Optional - get a free token at [mapbox.com](https://mapbox.com) for 3D globe visualization |
| `DISABLE_SIGNUP` | Set to `true` after creating your admin account |

## 1.4 Cloudflare Tunnel

Add a public hostname in Cloudflare Zero Trust:

| Public hostname | Service |
|----------------|---------|
| `rybbit.example.com` | `http://rybbit_nginx:80` |

> This setup uses nginx to route `/api` requests to the backend and all other requests to the frontend. This is required because Rybbit expects both services on the same domain.
{.is-info}

# 2 · First Login

1. Navigate to `https://rybbit.example.com/signup`
2. Create your admin account
3. Set `DISABLE_SIGNUP=true` in the backend environment and redeploy to prevent new signups

# 3 · Adding the Tracking Script

Once logged in, add a site and copy the tracking script to your website's `<head>` tag:

```html
<script
  src="https://rybbit.example.com/api/script.js"
  data-site-id="YOUR_SITE_ID"
  defer
></script>
```

> Replace `YOUR_SITE_ID` with the ID shown in your Rybbit dashboard.
{.is-info}
