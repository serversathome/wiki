---
title: Rybbit
description: A guide to deploying Rybbit
published: true
date: 2026-10-07T00:16:22.520Z
tags: 
editor: markdown
dateCreated: 2026-01-24T19:52:22.810Z
---

# <img src="/rybbit.png" class="tab-icon"> What is Rybbit?

**Rybbit** is an open source, privacy-first web analytics platform. It's a self-hosted alternative to Google Analytics that respects user privacy while providing detailed insights about your website traffic.

<div class="glance">
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>6 services</b></div>
  <div><span>Depends on</span><b>Postgres, ClickHouse and Redis</b></div>
  <div><span>Difficulty</span><b class="difficulty intermediate">Intermediate</b></div>
  <div class="glance-links">
    <a href="https://github.com/rybbit-io/rybbit"><i class="mdi mdi-github"></i>Project</a>
    <a href="https://rybbit.com/docs"><i class="mdi mdi-book-open-variant"></i>Docs</a>
  </div>
</div>

# <img src="/cloudflare.png" class="tab-icon"> Cloudflare Tunnel Setup

This guide is for deploying Rybbit behind a **Cloudflare Tunnel** reverse proxy. Rybbit requires path-based routing (`/api` and a few `/.well-known` paths go to the backend, everything else goes to the frontend), which the Cloudflare Zero Trust dashboard doesn't support natively. To work around this, we use an nginx container to handle the routing.

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

## 1.1 Create the Folders

Create the data folders and give them to the TrueNAS apps user, since every stateful container in this stack runs as `568:568`:

```bash
mkdir -p /mnt/tank/configs/rybbit/{clickhouse-data,postgres-data,redis-data}
chown -R 568:568 /mnt/tank/configs/rybbit
```

> **Upgrading an older Rybbit install?** Run the `chown` above against your existing `clickhouse-data` and `postgres-data` folders before redeploying, or ClickHouse and Postgres will fail to start with permission errors.
{.is-warning}

## 1.2 Create the Nginx Config

Create the nginx configuration file at `/mnt/tank/configs/rybbit/nginx.conf`:

```nginx
pid /tmp/nginx.pid;

events {
    worker_connections 1024;
}

http {
    client_body_temp_path /tmp/client_temp;
    proxy_temp_path       /tmp/proxy_temp;
    fastcgi_temp_path     /tmp/fastcgi_temp;
    uwsgi_temp_path       /tmp/uwsgi_temp;
    scgi_temp_path        /tmp/scgi_temp;

    server {
        listen 8080;

        # Backend API (no trailing slash, no rewrite since Rybbit v1.0)
        location /api/ {
            proxy_pass http://rybbit_backend:3001;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # OAuth discovery endpoints used by the Rybbit MCP server
        location ~ ^/\.well-known/(oauth-|openid-configuration) {
            proxy_pass http://rybbit_backend:3001;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # Frontend
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

> Nginx listens on `8080` and keeps its pid and temp files in `/tmp` so it can run as the `568` apps user instead of root.
{.is-info}

## 1.3 Deploy the Stack

```yaml
services:
  rybbit_clickhouse:
    image: clickhouse/clickhouse-server:26.3.17.4
    container_name: rybbit_clickhouse
    user: "568:568"
    volumes:
      - /mnt/tank/configs/rybbit/clickhouse-data:/var/lib/clickhouse
    configs:
      - source: clickhouse_network
        target: /etc/clickhouse-server/config.d/network.xml
      - source: clickhouse_logging
        target: /etc/clickhouse-server/config.d/logging_rules.xml
      - source: clickhouse_resource_limits
        target: /etc/clickhouse-server/config.d/resource_limits.xml
      - source: clickhouse_user_settings
        target: /etc/clickhouse-server/users.d/user_settings.xml
    environment:
      - CLICKHOUSE_DB=analytics
      - CLICKHOUSE_USER=default
      - CLICKHOUSE_PASSWORD=changeme
      - CLICKHOUSE_DEFAULT_ACCESS_MANAGEMENT=1
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
    user: "568:568"
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

  rybbit_redis:
    image: redis:8.6.4-alpine
    container_name: rybbit_redis
    user: "568:568"
    volumes:
      - /mnt/tank/configs/rybbit/redis-data:/data
    command:
      - redis-server
      - --requirepass
      - changeme
      - --appendonly
      - "yes"
      - --appendfsync
      - everysec
      - --maxmemory-policy
      - noeviction
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "changeme", "--no-auth-warning", "ping"]
      interval: 3s
      timeout: 5s
      retries: 5
      start_period: 5s
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
      - REDIS_HOST=rybbit_redis
      - REDIS_PORT=6379
      - REDIS_PASSWORD=changeme
      - BETTER_AUTH_SECRET=generate_with_openssl_rand_hex_32
      - BASE_URL=https://rybbit.example.com
      - DISABLE_SIGNUP=false
      - DISABLE_TELEMETRY=true
      - MAPBOX_TOKEN=optional_for_globe_visualization
    depends_on:
      rybbit_clickhouse:
        condition: service_healthy
      rybbit_postgres:
        condition: service_started
      rybbit_redis:
        condition: service_healthy
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
    user: "568:568"
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

configs:
  clickhouse_network:
    content: |
      <clickhouse>
          <listen_host>0.0.0.0</listen_host>
      </clickhouse>

  clickhouse_logging:
    content: |
      <clickhouse>
        <logger>
            <level>warning</level>
            <console>true</console>
        </logger>
        <query_thread_log remove="remove"/>
        <query_log remove="remove"/>
        <query_views_log remove="remove"/>
        <query_metric_log remove="remove"/>
        <error_log remove="remove"/>
        <opentelemetry_span_log remove="remove"/>
        <text_log remove="remove"/>
        <trace_log remove="remove"/>
        <metric_log remove="remove"/>
        <asynchronous_metric_log remove="remove"/>
        <session_log remove="remove"/>
        <part_log remove="remove"/>
        <latency_log remove="remove"/>
        <processors_profile_log remove="remove"/>
      </clickhouse>

  clickhouse_resource_limits:
    content: |
      <clickhouse>
        <max_server_memory_usage_to_ram_ratio>0.80</max_server_memory_usage_to_ram_ratio>
        <concurrent_threads_soft_limit_ratio_to_cores>1</concurrent_threads_soft_limit_ratio_to_cores>
        <merges_mutations_memory_usage_to_ram_ratio>0.25</merges_mutations_memory_usage_to_ram_ratio>
      </clickhouse>

  clickhouse_user_settings:
    content: |
      <clickhouse>
        <profiles>
          <default>
            <enable_json_type>1</enable_json_type>
            <async_insert>1</async_insert>
            <wait_for_async_insert>1</wait_for_async_insert>
            <log_queries>0</log_queries>
            <log_query_threads>0</log_query_threads>
            <log_processors_profiles>0</log_processors_profiles>
            <max_memory_usage>32000000000</max_memory_usage>
            <max_threads>16</max_threads>
          </default>
        </profiles>
      </clickhouse>
```

> Replace all `changeme` passwords with secure values. They have to match in pairs: `CLICKHOUSE_PASSWORD` between clickhouse and backend, `POSTGRES_USER`/`POSTGRES_PASSWORD` between postgres and backend, and the Redis password in **three** places (the `--requirepass` value, the healthcheck, and `REDIS_PASSWORD` in the backend).
{.is-warning}

> The `configs:` block at the bottom uses inline config content, which needs Docker Compose v2.23.1 or newer. Current TrueNAS and Dockge both meet this. ClickHouse and Redis memory settings come straight from the upstream compose file: ClickHouse is capped at 80% of host RAM and Redis uses `noeviction` so session data is never dropped.
{.is-info}

> If you named your tunnel network something other than `cftunnel`, update the network name in the `networks:` block.
{.is-info}

## 1.4 Configuration

| Variable | Description |
|----------|-------------|
| `BETTER_AUTH_SECRET` | Generate with `openssl rand -hex 32` |
| `BASE_URL` | Your full domain with https (e.g., `https://rybbit.example.com`) |
| `NEXT_PUBLIC_BACKEND_URL` | Same as `BASE_URL` |
| `REDIS_PASSWORD` | Must match the `--requirepass` value on the Redis container |
| `MAPBOX_TOKEN` | Optional. Get a free token at [mapbox.com](https://mapbox.com) for 3D globe visualization |
| `DISABLE_SIGNUP` | Set to `true` (on both backend and client) after creating your admin account |
| `DISABLE_TELEMETRY` | Set to `true` to stop Rybbit sending anonymous usage telemetry |
{.dense}

> Redis is now a required dependency. It backs session tracking, user identity, bot detection counters and background job queues, and the backend will not start without it.
{.is-info}

## 1.5 Cloudflare Tunnel

Add a public hostname in Cloudflare Zero Trust:

| Public hostname | Service |
|----------------|---------|
| `rybbit.example.com` | `http://rybbit_nginx:8080` |

> If you are upgrading from the old version of this guide, change the tunnel service from port `80` to port `8080`.
{.is-warning}

# 2 · First Login

1. Navigate to `https://rybbit.example.com/signup`
2. Create your admin account
3. Set `DISABLE_SIGNUP=true` on the backend and `NEXT_PUBLIC_DISABLE_SIGNUP=true` on the client, then redeploy to prevent new signups

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