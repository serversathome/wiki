---
title: Penpot
description: A guide to deploying Penpot
published: true
date: 2026-01-15T15:30:41.906Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:06.342Z
---

# ![](/penpot.png){class="tab-icon"} What is Penpot?


Penpot is the first open-source design tool for design and code collaboration. Designers can create stunning designs, interactive prototypes, design systems at scale, while developers enjoy ready-to-use code and make their workflow easy and fast. And all of this with no handoff drama.


# 1 · Deploy Penpot
# {.tabset}
## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Set the **Public URI** to the IP:Port of your server
1. Set a **Database Password**
1. Set a **Redis Password**
1. Set the **Penpot Assets Storage** to **Host Path**
1. Set the **Penpot Postgres Data Storage** to **Host Path**

> Remember to check the box for **Automatic Permissions** under **Postgres Data Storage**
{.is-warning}


## <img src="/docker.png" class="tab-icon"> Docker Compose


```yaml

x-flags: &penpot-flags
  PENPOT_FLAGS: disable-email-verification enable-smtp enable-prepl-server disable-secure-session-cookies

x-uri: &penpot-public-uri
  PENPOT_PUBLIC_URI: http://localhost:9001

x-body-size: &penpot-http-body-size
  PENPOT_HTTP_SERVER_MAX_BODY_SIZE: 31457280
  PENPOT_HTTP_SERVER_MAX_MULTIPART_BODY_SIZE: 367001600


services:
  penpot-frontend:
    image: penpotapp/frontend:latest
    restart: unless-stopped
    ports:
      - 9001:8080

    volumes:
      - /mnt/tank/configs/penpot/penpot_assets:/opt/data/assets

    depends_on:
      - penpot-backend
      - penpot-exporter

    environment:
      << : [*penpot-flags, *penpot-http-body-size]

  penpot-backend:
    image: penpotapp/backend:latest
    restart: unless-stopped

    volumes:
      - /mnt/tank/configs/penpot/penpot_assets:/opt/data/assets

    depends_on:
      penpot-postgres:
        condition: service_healthy
      penpot-valkey:
        condition: service_healthy

    environment:
      << : [*penpot-flags, *penpot-public-uri, *penpot-http-body-size]

      PENPOT_SECRET_KEY: my-insecure-key

      PENPOT_PREPL_HOST: 0.0.0.0

      PENPOT_DATABASE_URI: postgresql://penpot-postgres/penpot
      PENPOT_DATABASE_USERNAME: penpot
      PENPOT_DATABASE_PASSWORD: penpot
     
      PENPOT_REDIS_URI: redis://penpot-valkey/0
      
      PENPOT_ASSETS_STORAGE_BACKEND: assets-fs
      PENPOT_STORAGE_ASSETS_FS_DIRECTORY: /opt/data/assets

    
      PENPOT_TELEMETRY_ENABLED: false
      PENPOT_TELEMETRY_REFERER: compose

     
      PENPOT_SMTP_DEFAULT_FROM: no-reply@example.com
      PENPOT_SMTP_DEFAULT_REPLY_TO: no-reply@example.com
      PENPOT_SMTP_HOST: penpot-mailcatch
      PENPOT_SMTP_PORT: 1025
      PENPOT_SMTP_USERNAME:
      PENPOT_SMTP_PASSWORD:
      PENPOT_SMTP_TLS: false
      PENPOT_SMTP_SSL: false

  penpot-exporter:
    image: penpotapp/exporter:latest
    restart: unless-stopped

    depends_on:
      penpot-valkey:
        condition: service_healthy

    environment:

      PENPOT_PUBLIC_URI: http://penpot-frontend:8080
      PENPOT_REDIS_URI: redis://penpot-valkey/0

  penpot-postgres:
    image: "postgres:15"
    restart: unless-stopped
    stop_signal: SIGINT

    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U penpot"]
      interval: 2s
      timeout: 10s
      retries: 5
      start_period: 2s

    volumes:
      - /mnt/tank/configs/penpot/penpot_postgres_v15:/var/lib/postgresql/data

    environment:
      - POSTGRES_INITDB_ARGS=--data-checksums
      - POSTGRES_DB=penpot
      - POSTGRES_USER=penpot
      - POSTGRES_PASSWORD=penpot

  penpot-valkey:
    image: valkey/valkey:8.1
    restart: unless-stopped

    healthcheck:
      test: ["CMD-SHELL", "valkey-cli ping | grep PONG"]
      interval: 1s
      timeout: 3s
      retries: 5
      start_period: 3s


  penpot-mailcatch:
    image: sj26/mailcatcher:latest
    restart: unless-stopped
    expose:
      - '1025'
    ports:
      - "1080:1080"

```