---
title: Reactive Resume
description: A guide to deploying Reactive Resume
published: true
date: 2026-01-15T15:31:19.229Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:58.027Z
---

# <img src="/reactive-resume.png" class="tab-icon"> What is Reactive Resume?
A free and open-source resume builder that simplifies the process of creating, updating, and sharing your resume.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Reactive Resume
```yaml
services:

  postgres:
    image: postgres:16-alpine
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/reactiveresume/postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: postgres
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  minio:
    image: minio/minio:latest
    restart: unless-stopped
    command: server /data
    ports:
      - "9000:9000"
    volumes:
      - /mnt/tank/configs/reactiveresume/minio_data:/data
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin

  chrome:
    image: ghcr.io/browserless/chromium:v2.18.0 # Upgrading to newer versions causes issues
    restart: unless-stopped
    extra_hosts:
      - "host.docker.internal:host-gateway"
    environment:
      TIMEOUT: 10000
      CONCURRENT: 10
      TOKEN: chrome_token
      EXIT_ON_HEALTH_FAILURE: "true"
      PRE_REQUEST_HEALTH_CHECK: "true"

  app:
    image: amruthpillai/reactive-resume:latest
    restart: unless-stopped
    ports:
      - "3000:3000"
    depends_on:
      - postgres
      - minio
      - chrome
    environment:
      # -- Environment Variables --
      PORT: 3000
      NODE_ENV: production

      # -- URLs --
      PUBLIC_URL: http://[your-server-ip]:3000
      STORAGE_URL: http://[your-server-ip]:9000/default

      # -- Printer (Chrome) --
      CHROME_TOKEN: chrome_token
      CHROME_URL: ws://chrome:3000

      # -- Database (Postgres) --
      DATABASE_URL: postgresql://postgres:postgres@postgres:5432/postgres

      # -- Auth --
      ACCESS_TOKEN_SECRET: access_token_secret
      REFRESH_TOKEN_SECRET: refresh_token_secret

      # -- Emails --
      MAIL_FROM: noreply@localhost
      # SMTP_URL: smtp://user:pass@smtp:587 # Optional

      # -- Storage (Minio) --
      STORAGE_ENDPOINT: minio
      STORAGE_PORT: 9000
      STORAGE_REGION: us-east-1 # Optional
      STORAGE_BUCKET: default
      STORAGE_ACCESS_KEY: minioadmin
      STORAGE_SECRET_KEY: minioadmin
      STORAGE_USE_SSL: "false"
      STORAGE_SKIP_BUCKET_CHECK: "false"
```
> Replace `[your-server-ip]` with the correct IP before deploying!
{.is-warning}


# 2 · Signing In
1. Navigate to the IP of the server on port `3000`
1. Click **Go to Dashboard**
1. Click **Create a new one** to create an admin account
1. Ignore the part about verifying your email and click **Go to Dashboard**

