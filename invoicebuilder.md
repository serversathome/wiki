---
title: Invoice-Builder
description: A guide to deploying Invoice Builder
published: true
date: 2026-03-02T14:57:26.804Z
tags: 
editor: markdown
dateCreated: 2026-03-02T14:57:26.804Z
---

# What is Invoice Builder?

**Invoice Builder** is an offline-first, open-source invoicing and quoting application designed for freelancers and small businesses who want full control over their data. It supports creating and managing invoices and quotes with PDF export, multi-currency support, customizable templates, and a local SQLite database — no accounts, no cloud, no subscriptions required.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Invoice Builder

```yaml
services:
  backend:
    image: ghcr.io/piratuks/invoice-builder:latest
    extra_hosts:
      - 'host.docker.internal:host-gateway'
    container_name: invoice-builder-backend
    environment:
      - SERVICE=backend
      - NODE_ENV=docker
      - DB_DIRECTORY=/data
      - PORT=3000
      - DEV_SERVER_URL=0.0.0.0
      - FE_SERVER_URL=http://localhost:3001
      - MIGRATIONS_PATH=/app/dist-be/backend/server/shared/migrations
    volumes:
      - /mnt/tank/configs/invoice-builder:/data
    healthcheck:
      test:
        [
          'CMD',
          'node',
          '-e',
          "const net=require('net');const c=net.createConnection(3000,'localhost');c.on('connect',()=>{c.destroy();process.exit(0)});c.on('error',()=>process.exit(1))"
        ]
      interval: 5s
      timeout: 3s
      retries: 10
      start_period: 10s
    restart: unless-stopped
  frontend:
    image: ghcr.io/piratuks/invoice-builder:latest
    container_name: invoice-builder-frontend
    ports:
      - '3001:3001'
    environment:
      - SERVICE=frontend
      - BACKEND_HOST=backend
      - BACKEND_PORT=3000
    depends_on:
      backend:
        condition: service_healthy
    restart: unless-stopped
```

# 2 · Initial Setup

Once the containers are running and you access the web UI:

1. **Create or open a database** — Invoice Builder uses a local database file you own
2. **Add a Business** — your company details for invoices
3. **Add a Currency** — select the currencies you work with
4. **Add a Client** — your customer information
5. **Add Items** — products or services you bill for
6. **Create an Invoice or Quote** — preview and export to PDF

## 2.1 PDF Customization

Invoice Builder offers extensive PDF customization:

- A4 and Letter page formats
- Layout presets with color, font size, and logo size controls
- Table header and row styling
- Invoice and quote watermarks (including paid watermark)
- Signature support — upload or hand-draw signatures
- Style profiles for consistent theming
- Attachments — include images in PDFs
- Custom header sections and custom values in item tables

# <img src="/youtube.png" class="tab-icon"> 3 · Video

*Video coming soon — check back later!*