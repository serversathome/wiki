---
title: Colanode
description: A guide to deploying Colanode
published: true
date: 2026-01-12T10:32:27.710Z
tags: 
editor: markdown
dateCreated: 2025-12-30T01:46:10.392Z
---

# <img src="/colanode.png" class="tab-icon"> What is Colanode?

Open-source and local-first Slack and Notion alternative that puts you in control of your data. 

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Colanode
```yaml
services:
  postgres:
    image: pgvector/pgvector:pg17
    container_name: colanode_postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: colanode_user
      POSTGRES_PASSWORD: postgrespass123
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
      - your_valkey_password
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
      SERVER_NAME: Colanode Local
      SERVER_AVATAR: ""
      SERVER_MODE: standalone
      ACCOUNT_VERIFICATION_TYPE: automatic
      ACCOUNT_OTP_TIMEOUT: "600"
      USER_STORAGE_LIMIT: "10737418240"
      USER_MAX_FILE_SIZE: "104857600"
      POSTGRES_URL: postgres://colanode_user:postgrespass123@postgres:5432/colanode_db
      REDIS_URL: redis://:your_valkey_password@valkey:6379/0
      REDIS_DB: "0"
      STORAGE_TYPE: file
      STORAGE_FILE_DIRECTORY: /var/lib/colanode/storage
      SMTP_ENABLED: "false"
      # For local-only, CORS can be your web UI origin:
      SERVER_CORS_ORIGIN: http://localhost:4000 #add your server IP
    ports:
      - 3000:3000
    volumes:
      - /mnt/tank/configs/colanode/server_storage:/var/lib/colanode/storage
  web:
    image: ghcr.io/colanode/web:latest
    container_name: colanode_web
    restart: unless-stopped
    ports:
      - 4000:80

```

# 2 · Setting Up

1. Navigate to `http://IP:4000`
1. Select **Add Sever** in the **Registration** section. To know what to enter for the address, click the `3000` pill in dockge to get the server address.

# <img src="/youtube.png" class="tab-icon"> 3 · Video
https://youtu.be/fhdE2zLmkAQ
