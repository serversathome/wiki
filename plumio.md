---
title: Plumio
description: A guide to deploying Plumio
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-03-02T21:21:42.589Z
---

# <img src="/plumio.png" class="tab-icon"> What is Plumio?

**Plumio** is a self-hosted markdown editor with live preview, end-to-end document encryption, multi-user support, and multi-organization capabilities. It's designed for individuals and teams who want a secure, private, and customizable note-taking solution with full control over their data. The frontend is built with SolidJS, and the backend runs on HonoJS (Node.js) with a lightweight SQLite database.

<div class="glance">
  <div><span>Port</span><b><code>3000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/albertasaftei/plumio"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Plumio
 On Dockge, use the below compose.yaml file:



```yaml
services:
  plumio:
    image: ghcr.io/albertasaftei/plumio:latest
    container_name: plumio
    restart: unless-stopped
    ports:
      - "3000:3000"
      - "3001:3001"
    environment:
      - NODE_ENV=production
      - BACKEND_INTERNAL_PORT=3001
      - DOCUMENTS_PATH=/data/documents
      - DB_PATH=/data/plumio.db
      - JWT_SECRET=${JWT_SECRET}
      - ENCRYPTION_KEY=${ENCRYPTION_KEY}
      - ALLOWED_ORIGINS=http://YOUR_SERVER_IP:3000
      - ENABLE_ENCRYPTION=true
    volumes:
      - /mnt/tank/configs/plumio/data:/data
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3001/api/health"]
      interval: 5m
      timeout: 10s
      retries: 3
      start_period: 40s
    deploy:
      resources:
        limits:
          memory: 1G
        reservations:
          memory: 512M
```

Replace `YOUR_SERVER_IP` in `ALLOWED_ORIGINS` with your actual server IP or hostname


And in the ENV section below the compose window add the following content:

```env
JWT_SECRET=
ENCRYPTION_KEY=
```

> 
> Generate `JWT_SECRET` with `openssl rand -base64 32`.
> Generate `ENCRYPTION_KEY` with `openssl rand -hex 32`. It must be exactly 64 hexadecimal characters.
{.is-info}

> Back up your `ENCRYPTION_KEY`. If you change or lose it, your encrypted documents can no longer be read.
{.is-danger}



After about 30 seconds, navigate to `http://your-server-ip:3000` to access Plumio.

> 
> Port `3000` serves the frontend UI and port `3001` serves the backend API. Both must be exposed.
{.is-info}

> 
> Check the logs with `docker compose logs` if anything isn't working as expected.
{.is-warning}

