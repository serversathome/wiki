---
title: Colanode
description: A guide to deploying Colanode
published: true
date: '2026-11-04T16:23:24.993Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:03:52.103Z'
---

# What is Colanode?

Open-source and local-first Slack and Notion alternative that puts you in control of your data. 

# :logos:docker-icon: 1 · Deploy Colanode

Create `/mnt/tank/configs/colanode/config.json` first, replacing `your-server-ip` with your server's IP:

```json
{
  "cors": { "origin": "http://your-server-ip:4000" },
  "storage": { "provider": { "type": "file", "directory": "/var/lib/colanode/storage" } }
}
```

```yaml
services:
  postgres:
    image: pgvector/pgvector:pg17
    container_name: colanode_postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: colanode_user
      POSTGRES_PASSWORD: CHANGEME_DB_PASSWORD
      POSTGRES_DB: colanode_db
    volumes:
      - /mnt/tank/configs/colanode/postgres_data:/var/lib/postgresql/data

  valkey:
    image: valkey/valkey:8.1
    container_name: colanode_valkey
    restart: unless-stopped
    command:
      - valkey-server
      - --requirepass
      - CHANGEME_VALKEY_PASSWORD
    volumes:
      - /mnt/tank/configs/colanode/valkey_data:/data

  server:
    image: ghcr.io/colanode/server:latest
    container_name: colanode_server
    restart: unless-stopped
    depends_on:
      - postgres
      - valkey
    environment:
      NODE_ENV: production
      POSTGRES_URL: postgres://colanode_user:CHANGEME_DB_PASSWORD@postgres:5432/colanode_db
      REDIS_URL: redis://:CHANGEME_VALKEY_PASSWORD@valkey:6379/0
      CONFIG: /config.json
    ports:
      - 3000:3000
    volumes:
      - /mnt/tank/configs/colanode/server_storage:/var/lib/colanode/storage
      - /mnt/tank/configs/colanode/config.json:/config.json:ro

  web:
    image: ghcr.io/colanode/web:latest
    container_name: colanode_web
    restart: unless-stopped
    ports:
      - 4000:80
```

1. Generate two passwords with `openssl rand -hex 24`. Replace both `CHANGEME_DB_PASSWORD` values with the first and both `CHANGEME_VALKEY_PASSWORD` values with the second
2. Deploy the stack

> [!WARNING]
> Upgrading from pre-0.4? Move your settings from env vars into `config.json`; see the [v0.4.0 release notes](https://github.com/colanode/colanode/releases/tag/v0.4.0).

# 2 · Setting Up

1. Navigate to `http://IP:4000`
1. Select **Add Sever** in the **Registration** section. To know what to enter for the address, click the `3000` pill in dockge to get the server address.

# :logos:youtube-icon: 3 · Video
::block-youtube{url="https://youtu.be/fhdE2zLmkAQ"}
::

