---
title: Scanopy
description: A guide to deploying Scanopy
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:08:14.022Z
---

# <img src="/scanopy.png" class="tab-icon"> What is Scanopy?

Scanopy scans your network, identifies hosts and services, and generates an interactive visualization showing how everything connects, letting you easily create and maintain network documentation.

<div class="glance">
  <div><span>Port</span><b><code>60072</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>3 services</b></div>
  <div><span>Depends on</span><b>Postgres</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/scanopy/scanopy"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Scanopy
```yaml
name: scanopy

services:
  daemon:
    image: ghcr.io/scanopy/scanopy/daemon:latest
    container_name: scanopy-daemon
    network_mode: host
    privileged: true
    restart: unless-stopped
    environment:
      SCANOPY_LOG_LEVEL: ${SCANOPY_LOG_LEVEL:-info}
      SCANOPY_SERVER_URL: http://127.0.0.1:60072
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:60073/api/health || exit 1"]
      interval: 5s
      timeout: 3s
      retries: 15
    volumes:
      - /mnt/tank/configs/scanopy/daemon-config:/root/.config/scanopy/daemon
      # Comment out the line below to disable docker discovery
      - /var/run/docker.sock:/var/run/docker.sock:ro

  postgres:
    image: postgres:17-alpine
    environment:
      POSTGRES_DB: scanopy
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-password}
    volumes:
      - /mnt/tank/configs/scanopy/postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped
    networks:
      - scanopy

  server:
    image: ghcr.io/scanopy/scanopy/server:latest
    ports:
      - "60072:60072"
    # Passes an optional .env (SMTP and other SCANOPY_* settings) straight to the server
    env_file:
      - path: .env
        required: false
    environment:
      SCANOPY_LOG_LEVEL: ${SCANOPY_LOG_LEVEL:-info}
      SCANOPY_DATABASE_URL: postgresql://postgres:${POSTGRES_PASSWORD:-password}@postgres:5432/scanopy
      SCANOPY_WEB_EXTERNAL_PATH: /app/static
      SCANOPY_PUBLIC_URL: ${SCANOPY_PUBLIC_URL:-http://localhost:60072}
      # How the server reaches the integrated daemon (running on the host network)
      SCANOPY_INTEGRATED_DAEMON_URL: http://host.docker.internal:60073
    volumes:
      - /mnt/tank/configs/scanopy/data:/data
    extra_hosts:
      - "host.docker.internal:host-gateway"
    depends_on:
      postgres:
        condition: service_healthy
      daemon:
        condition: service_started
    restart: unless-stopped
    networks:
      - scanopy

networks:
  scanopy:
    driver: bridge
    ipam:
      config:
        - subnet: 172.31.0.0/16
          gateway: 172.31.0.1
```

> The daemon now keeps its config in `/root/.config/scanopy/daemon`. If you deployed with the older `/root/.config/daemon` path, update the volume line, or the daemon re-registers itself every time the container is recreated.
{.is-info}

# 2 · Access Scanopy
Open `http://IP:60072` and create your account. The first network discovery starts on its own.