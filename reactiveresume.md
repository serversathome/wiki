---
title: Reactive Resume
description: A guide to deploying Reactive Resume
published: true
date: 2026-01-25T11:59:52.944Z
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
    image: postgres:latest
    restart: unless-stopped
    environment:
      POSTGRES_DB: postgres
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - /mnt/tank/configs/postgres_data:/var/lib/postgresql
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres", "-d", "postgres"]
      start_period: 10s
      interval: 30s
      timeout: 10s
      retries: 3

  browserless:
    image: ghcr.io/browserless/chromium:latest
    restart: unless-stopped
    environment:
      - QUEUED=30
      - HEALTH=true
      - CONCURRENT=20
      - TOKEN=1234567890
    healthcheck:
      test:
        ["CMD", "curl", "-f", "http://localhost:3000/pressure?token=1234567890"]
      interval: 10s
      timeout: 5s
      retries: 10


  seaweedfs:
    image: chrislusf/seaweedfs:latest
    restart: unless-stopped
    command: server -s3 -filer -dir=/data -ip=0.0.0.0
    environment:
      - AWS_ACCESS_KEY_ID=seaweedfs
      - AWS_SECRET_ACCESS_KEY=seaweedfs
    volumes:
      - /mnt/tank/configs/seaweedfs_data:/data
    healthcheck:
      test: ["CMD", "wget", "-q", "-O", "/dev/null", "http://localhost:8888"]
      start_period: 10s
      interval: 30s
      timeout: 10s
      retries: 3

  seaweedfs-create-bucket:
    image: quay.io/minio/mc:latest
    restart: on-failure
    entrypoint: >
      /bin/sh -c "
        sleep 5;
        mc alias set seaweedfs http://seaweedfs:8333 seaweedfs seaweedfs;
        mc mb seaweedfs/reactive-resume;
        exit 0;
      "
    depends_on:
      seaweedfs:
        condition: service_healthy

  app:
    image: amruthpillai/reactive-resume:latest
    # image: ghcr.io/amruthpillai/reactive-resume:latest
    environment:
      # Server
      - TZ=Etc/UTC
      - NODE_ENV=production
      - APP_URL=http://[your-server-ip]:3000
      - PRINTER_APP_URL=http://app:3000
      # Printer
      - PRINTER_ENDPOINT=ws://browserless:3000?token=1234567890
      # - PRINTER_ENDPOINT=http://chrome:9222 # Or, if you're using chromedp/headless-shell
      # Database
      - DATABASE_URL=postgresql://postgres:postgres@postgres:5432/postgres
      # Authentication
      - AUTH_SECRET=change-me-to-a-secure-secret-key-in-production
      # Storage
      - S3_ACCESS_KEY_ID=seaweedfs
      - S3_SECRET_ACCESS_KEY=seaweedfs
      - S3_ENDPOINT=http://seaweedfs:8333
      - S3_BUCKET=reactive-resume
      - S3_FORCE_PATH_STYLE=true
    volumes:
      - /mnt/tank/configs/reactive_resume_data:/app/data
    ports:
      - "3000:3000"
    depends_on:
      postgres:
        condition: service_healthy
      browserless:
        condition: service_healthy
      seaweedfs-create-bucket:
        condition: service_completed_successfully
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/api/health"]
      start_period: 10s
      interval: 30s
      timeout: 10s
      retries: 3
```
> Replace `[your-server-ip]` with the correct IP before deploying!
{.is-warning}


# 2 · Signing In
1. Navigate to the IP of the server on port `3000`
1. Click **Go to Dashboard**
1. Click **Create a new one** to create an admin account
1. Ignore the part about verifying your email and click **Go to Dashboard**

