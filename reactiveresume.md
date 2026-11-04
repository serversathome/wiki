---
title: Reactive Resume
description: A guide to deploying Reactive Resume
published: true
date: '2026-11-04T16:23:11.616Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:07:58.027Z'
---

# What is Reactive Resume?
A free and open-source resume builder that simplifies the process of creating, updating, and sharing your resume.

# :logos:docker-icon: 1 · Deploy Reactive Resume
Generate an `AUTH_SECRET` and a database password (run it twice):

```bash
openssl rand -hex 32
```

```yaml
services:
  postgres:
    image: postgres:18
    restart: unless-stopped
    environment:
      POSTGRES_DB: postgres
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: your_db_password
    volumes:
      - /mnt/tank/configs/postgres_data:/var/lib/postgresql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d postgres"]
      interval: 10s
      timeout: 5s
      retries: 10

  app:
    image: amruthpillai/reactive-resume:latest
    # image: ghcr.io/reactive-resume/reactive-resume:latest
    restart: unless-stopped
    environment:
      - TZ=America/New_York
      - APP_URL=http://[your-server-ip]:3000
      - DATABASE_URL=postgresql://postgres:your_db_password@postgres:5432/postgres
      - AUTH_SECRET=your_generated_secret
    volumes:
      - /mnt/tank/configs/reactive_resume_data:/app/data
    ports:
      - "3000:3000"
    depends_on:
      postgres:
        condition: service_healthy
```
> [!WARNING]
> Replace `[your-server-ip]` with the correct IP before deploying, and use the same password in `POSTGRES_PASSWORD` and `DATABASE_URL`!

> [!NOTE]
> Reactive Resume v6 renders PDFs itself and stores uploads in `/app/data`, so the old Browserless printer and SeaweedFS/bucket containers are no longer needed. (The `quay.io/minio/mc` image the old bucket job used can't be pulled anonymously anymore, which stopped the app from starting.) If you want S3 storage, Redis or AI features, see the upstream [environment variable reference](https://github.com/AmruthPillai/Reactive-Resume/blob/main/docs/self-hosting/environment-variables.mdx).

> [!WARNING]
> **Upgrading from v5?** v6 upgrades the database, and v5 can't run against it afterwards. Back up `/mnt/tank/configs/postgres_data` (and your SeaweedFS data if you used the old compose) before pulling, and follow the [upgrading to v6 guide](https://github.com/AmruthPillai/Reactive-Resume/blob/main/docs/self-hosting/upgrading-to-v6.mdx), which also covers converting older resume styles. If your existing uploads are in SeaweedFS, keep the `seaweedfs` service and the `S3_*` variables, and switch the bucket job to the `amazon/aws-cli` image as in the upstream [compose.yml](https://github.com/AmruthPillai/Reactive-Resume/blob/main/compose.yml).


# 2 · Signing In
1. Navigate to the IP of the server on port `3000`
1. Click **Go to Dashboard**
1. Click **Create a new one** to create an admin account
1. Ignore the part about verifying your email and click **Go to Dashboard**


