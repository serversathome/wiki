---
title: Mediqux
description: A guide to deploy Mediqux
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:06:26.492Z
---

# 🏥 What is Mediqux?
A privacy-first, locally-hosted medical record system for individuals, families, and small healthcare practices - featuring comprehensive patient management, lab result processing, and medication tracking with complete data sovereignty. 

<div class="glance">
  <div><span>Port</span><b><code>8086</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>3 services</b></div>
  <div><span>Depends on</span><b>Postgres</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/dmjoh/mediqux"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Mediqux
```yaml
services:
  postgres:
    image: postgres:17-alpine
    container_name: mediqux_postgres
    environment:
      POSTGRES_DB: mediqux
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: admin
    volumes:
      - /mnt/tank/configs/mediqux/db:/var/lib/postgresql/data
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin -d mediqux"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    image: ghcr.io/dmjoh/mediqux/mediqux-backend:latest
    container_name: mediqux_backend
    environment:
      NODE_ENV: production
      DB_HOST: postgres
      DB_PORT: 5432
      DB_NAME: mediqux
      DB_USER: admin
      DB_PASSWORD: admin
      JWT_SECRET: choose-a-new-secret # generate with: openssl rand -hex 32
      MAX_FILE_SIZE: 10MB
      UPLOAD_PATH: /app/uploads
      PUID: 1000
      PGID: 1000
      TRUST_PROXY_HOPS: 1 # number of reverse proxies in front of Mediqux
    volumes:
      - /mnt/tank/configs/mediqux/uploads:/app/uploads
    depends_on:
      postgres:
        condition: service_healthy
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "node", "-e", "require('http').get('http://localhost:3000/api/health', (res) => process.exit(res.statusCode === 200 ? 0 : 1))"]
      interval: 30s
      timeout: 10s
      retries: 3

  frontend:
    image: ghcr.io/dmjoh/mediqux/mediqux-frontend:latest
    container_name: mediqux_frontend
    ports:
      - "8086:8080"
    depends_on:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:8080"]
      interval: 30s
      timeout: 10s
      retries: 3
```

> Since Mediqux 2.0 the frontend listens on port `8080` inside the container and proxies the API on the same origin, so the backend no longer needs a published port. `MEDIQUX_API_URL`, `BACKEND_URL` and the old port variables were removed. If you are upgrading from 1.x, update your compose to match the one above. See the [release notes](https://github.com/dmjoh/mediqux/releases).
{.is-warning}

> Navigate to `http://IP:8086`. Self-service sign-up only works for the first account you create.
{.is-info}