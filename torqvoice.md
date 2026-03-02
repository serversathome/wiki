---
title: Torqvoice
description: A guide to deploying Torqvoice
published: true
date: 2026-03-02T19:14:06.166Z
tags: 
editor: markdown
dateCreated: 2026-03-02T19:14:06.166Z
---

# <img src="/torqvoice.png" class="tab-icon"> What is Torqvoice?

**Torqvoice** is a self-hosted workshop management platform built for automotive service businesses. It replaces scattered tools with a single place to manage customers, vehicles, service records, quotes, invoicing, inventory, and billing — all with a clean, modern UI. Whether you run one garage or multiple locations, Torqvoice provides a unified dashboard to keep your operations organized.


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Torqvoice

```yaml
services:
  torqvoice:
    image: ghcr.io/torqvoice/torqvoice:latest
    container_name: torqvoice
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://torqvoice:torqvoice@torqvoice-db:5432/torqvoice
      - BETTER_AUTH_SECRET=change-me-run-openssl-rand-hex-32
      - NEXT_PUBLIC_APP_URL=http://localhost:3000
    volumes:
      - /mnt/tank/configs/torqvoice/uploads:/app/data/uploads
    depends_on:
      torqvoice-db:
        condition: service_healthy

  torqvoice-db:
    image: postgres:16-alpine
    container_name: torqvoice-db
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/torqvoice/db:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: torqvoice
      POSTGRES_PASSWORD: torqvoice
      POSTGRES_DB: torqvoice
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U torqvoice"]
      interval: 5s
      timeout: 5s
      retries: 5
```

> Before deploying, generate a proper auth secret and replace the `BETTER_AUTH_SECRET` placeholder:
> ```bash
> openssl rand -hex 32
> ```
<!-- {blockquote:.is-warning} -->



# 2 · Configuration

## 2.1 Environment Variables

| Variable | Required | Description |
|---|---|---|
| `DATABASE_URL` | Yes | PostgreSQL connection string |
| `BETTER_AUTH_SECRET` | Yes | Auth secret — generate with `openssl rand -hex 32` |
| `NEXT_PUBLIC_APP_URL` | Yes | The full URL where Torqvoice is accessible |
{.dense}

## 2.2 Initial Setup

After logging in for the first time:

1. **Create a Company** — set your workshop name, address, and contact info
2. **Upload Your Logo** — add your company branding for invoices and quotes
3. **Customize Invoice Template** — adjust colors, fonts, and layout to match your brand
4. **Add Team Members** — invite staff and assign roles (owner, admin, member)
5. **Set Up Inventory** — add your parts catalog with supplier info and stock levels

## 2.3 Multi-Company Support

Torqvoice supports managing multiple workshops from a single login. To add another company, navigate to the company selector and create a new organization. Each company maintains its own separate data for customers, vehicles, work orders, and inventory.

## 2.4 Reverse Proxy

If placing Torqvoice behind a reverse proxy (e.g. Nginx Proxy Manager, Caddy, or Traefik), make sure to:

1. Set `NEXT_PUBLIC_APP_URL` to your public domain (e.g. `https://torqvoice.yourdomain.com`)
2. Forward traffic to port `3000`
3. Enable WebSocket support if your proxy requires it

> 
> Torqvoice generates shareable public links for invoices and quotes. The `NEXT_PUBLIC_APP_URL` must be correct for these links to work.
{.is-info}

## 2.5 Backups

The Torqvoice data is stored in two locations:

- **PostgreSQL database** — `/mnt/tank/configs/torqvoice/db`
- **File uploads** — `/mnt/tank/configs/torqvoice/uploads`

Make sure both directories are included in your backup strategy.

> 
> To back up the database manually, you can run:
> ```bash
> docker exec torqvoice-db pg_dump -U torqvoice torqvoice > torqvoice_backup.sql
> ```
{.is-success}

# <img src="/youtube.png" class="tab-icon"> 3 · Video

*Video coming soon — check back later!*