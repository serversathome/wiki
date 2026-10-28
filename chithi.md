---
title: Chithi
description: A guide to deploying Chithi
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-02-03T16:57:15.577Z
---

#  What is Chithi?

An end-to-end encrypted file sharing application with temporary links, user management, and background cleanup tasks.

<div class="glance">
  <div><span>Port</span><b><code>3000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>7 services</b></div>
  <div><span>Depends on</span><b>Postgres, Redis and RustFS</b></div>
  <div><span>Difficulty</span><b class="difficulty intermediate">Intermediate</b></div>
  <div class="glance-links">
    <a href="https://github.com/chithi-dev/chithi"><i class="mdi mdi-github"></i>Project</a>
    <a href="https://docs.chithi.dev"><i class="mdi mdi-book-open-variant"></i>Docs</a>
  </div>
</div>


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Chithi

Chithi now publishes pre-built images, so there is nothing to clone or build. This is upstream's "bring your own reverse proxy" deployment with TrueNAS paths.

```yaml
services:
  postgres:
    image: postgres:18
    container_name: chithi-db
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: CHANGEME_DB_PASSWORD
      POSTGRES_DB: chithi
    volumes:
      - /mnt/tank/configs/chithi/postgres:/var/lib/postgresql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5

  volume-permission-helper:
    image: alpine
    container_name: chithi-rustfs-volume-fix
    volumes:
      - /mnt/tank/configs/chithi/rustfs:/data
    command: >
      sh -c "
        chown -R 10001:10001 /data &&
        echo 'Volume Permissions fixed' &&
        exit 0
      "
    restart: "no"

  rustfs:
    image: rustfs/rustfs:1.0.0-alpha.85
    container_name: chithi-rustfs
    restart: unless-stopped
    user: "10001:10001"
    security_opt:
      - no-new-privileges:true
    volumes:
      - /mnt/tank/configs/chithi/rustfs:/data
    environment:
      RUSTFS_ADDRESS: "0.0.0.0:9000"
      RUSTFS_CONSOLE_ADDRESS: "0.0.0.0:9001"
      RUSTFS_VOLUMES: "/data/rustfs"
      RUSTFS_ACCESS_KEY: CHANGEME_RUSTFS_ACCESS_KEY
      RUSTFS_SECRET_KEY: CHANGEME_RUSTFS_SECRET_KEY
      RUSTFS_CONSOLE_ENABLE: "true"
    healthcheck:
      test:
        [
          "CMD",
          "sh",
          "-c",
          "curl -f http://localhost:9000/health && curl -f http://localhost:9001/rustfs/console/health",
        ]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    depends_on:
      volume-permission-helper:
        condition: service_completed_successfully

  redis:
    image: redis:8.6-alpine
    container_name: chithi-redis
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/chithi/redis:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s
      timeout: 10s
      retries: 3

  backend:
    image: ghcr.io/chithi-dev/chithi-backend:latest
    container_name: chithi-backend
    restart: unless-stopped
    command: /bin/sh /app/scripts/start_backend.sh
    ports:
      - 8000:8000
    environment: &backend-env
      POSTGRES_SERVER: postgres
      POSTGRES_PORT: 5432
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: CHANGEME_DB_PASSWORD
      POSTGRES_DB: chithi
      RUSTFS_ENDPOINT_URL: http://rustfs:9000
      RUSTFS_ACCESS_KEY: CHANGEME_RUSTFS_ACCESS_KEY
      RUSTFS_SECRET_ACCESS_KEY: CHANGEME_RUSTFS_SECRET_KEY
      CELERY_BROKER_URL: redis://redis:6379/0
      CELERY_RESULT_BACKEND: redis://redis:6379/0
      REDIS_ENDPOINT: redis://redis:6379/1
      ROOT_PATH: /api
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      rustfs:
        condition: service_healthy

  celery:
    image: ghcr.io/chithi-dev/chithi-backend:latest
    container_name: chithi-celery
    restart: unless-stopped
    command: /bin/sh /app/scripts/start_celery.sh
    environment: *backend-env
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      backend:
        condition: service_started

  frontend:
    image: ghcr.io/chithi-dev/chithi-frontend-node:latest
    container_name: chithi-frontend
    restart: unless-stopped
    ports:
      - 3000:3000
    environment:
      PUBLIC_BACKEND_API: https://chithi.yourdomain.com/api
    depends_on:
      backend:
        condition: service_started
```

1. Replace the `CHANGEME` values. Generate each one with `openssl rand -hex 24`. The database password and the two RustFS keys each appear twice and must match
2. Set `PUBLIC_BACKEND_API` to your domain followed by `/api`
3. Deploy the stack
4. Point your reverse proxy at it: requests to `/api/*` go to port `8000` with the `/api` prefix stripped, and everything else goes to port `3000`. The proxy must also add the cross-origin isolation headers. A Caddy example:

```
chithi.yourdomain.com {
    header {
        Cross-Origin-Opener-Policy "same-origin"
        Cross-Origin-Embedder-Policy "require-corp"
        Cross-Origin-Resource-Policy "same-origin"
    }

    handle_path /api/* {
        reverse_proxy your-server-ip:8000
    }

    handle {
        reverse_proxy your-server-ip:3000
    }
}
```

> Without the `Cross-Origin-Opener-Policy` and `Cross-Origin-Embedder-Policy` headers, Chithi's web workers will not load and encryption in the browser fails. Upstream also has ready-made versions of this setup with Caddy, NGINX or Traefik included in the [deployment docs](https://github.com/chithi-dev/chithi/tree/main/apps/docs/docs/deployments/docker/basic).
{.is-warning}

> Don't use the `docker-compose.yml` in the root of the Chithi repo; it only contains the databases, not the app.
{.is-warning}

# 2 · First Login

1. Access the web UI through your reverse proxy at `https://chithi.yourdomain.com`
2. Create your account through the onboarding flow

# <img src="/youtube.png" class="tab-icon"> 3 · Video

*No video yet — check back soon!*