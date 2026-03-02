---
title: Plumio
description: A guide to deploying Plumio
published: true
date: 2026-03-02T21:21:42.589Z
tags: 
editor: markdown
dateCreated: 2026-03-02T21:21:42.589Z
---

# <img src="/plumio.png" class="tab-icon"> What is Plumio?

**Plumio** is a self-hosted markdown editor with live preview, end-to-end document encryption, multi-user support, and multi-organization capabilities. It's designed for individuals and teams who want a secure, private, and customizable note-taking solution with full control over their data. The frontend is built with SolidJS, and the backend runs on HonoJS (Node.js) with a lightweight SQLite database.


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

1. Replace `YOUR_SERVER_IP` in `ALLOWED_ORIGINS` with your actual server IP or hostname
2. Deploy the stack:

And in the ENV section below the compose window add the following content:

```bash
JWT_SECRET=
ENCRYPTION_KEY=
VITE_API_URL=http://localhost:3001
```

> 
> Generate both secrets with: `openssl rand -base64 32`
> Run the command twice — once for `JWT_SECRET` and once for `ENCRYPTION_KEY`.
{.is-info}



After about 30 seconds, navigate to `http://your-server-ip:3000` to access Plumio.

> 
> Port `3000` serves the frontend UI and port `3001` serves the backend API. Both must be exposed.
{.is-info}

> 
> Check the logs with `docker compose logs` if anything isn't working as expected.
{.is-warning}

