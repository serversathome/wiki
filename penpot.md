---
title: Penpot
description: A guide to deploying Penpot
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:06.342Z
---

# ![](/penpot.png){class="tab-icon"} What is Penpot?


Penpot is the first open-source design tool for design and code collaboration. Designers can create stunning designs, interactive prototypes, design systems at scale, while developers enjoy ready-to-use code and make their workflow easy and fast. And all of this with no handoff drama.

<div class="glance">
  <div><span>Port</span><b><code>9001</code></b></div>
  <div><span>Deploy via</span><b>TrueNAS app or Docker compose</b></div>
  <div><span>Containers</span><b>8 services</b></div>
  <div><span>Depends on</span><b>Postgres and Valkey</b></div>
  <div><span>Difficulty</span><b class="difficulty intermediate">Intermediate</b></div>
</div>


# 1 · Deploy Penpot
# {.tabset}
## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Set the **Public URI** to the IP:Port of your server
1. Set a **Database Password**
1. Set a **Redis Password**
1. Set a **Secret Key** (generate one with `openssl rand -hex 32`)
1. Set the **Penpot Assets Storage** to **Host Path**
1. Set the **Penpot Postgres Data Storage** to **Host Path**

> Remember to check the box for **Automatic Permissions** under **Postgres Data Storage**
{.is-warning}


## <img src="/docker.png" class="tab-icon"> Docker Compose


Generate a secret key and paste it in place of `change-this-insecure-key`:

```bash
python3 -c "import secrets; print(secrets.token_urlsafe(64))"
```

```yaml
x-flags: &penpot-flags
  PENPOT_FLAGS: disable-email-verification enable-smtp enable-prepl-server disable-secure-session-cookies enable-mcp enable-admin-console

x-uri: &penpot-public-uri
  PENPOT_PUBLIC_URI: http://localhost:9001

x-body-size: &penpot-http-body-size
  PENPOT_HTTP_SERVER_MAX_BODY_SIZE: 367001600
  PENPOT_HTTP_SERVER_MAX_MULTIPART_BODY_SIZE: 367001600

x-secret-key: &penpot-secret-key
  PENPOT_SECRET_KEY: change-this-insecure-key

x-database: &penpot-database
  PENPOT_DATABASE_URI: postgresql://penpot-postgres/penpot
  PENPOT_DATABASE_USERNAME: penpot
  PENPOT_DATABASE_PASSWORD: penpot

services:
  penpot-frontend:
    image: penpotapp/frontend:2.18
    restart: unless-stopped
    ports:
      - 9001:8080

    volumes:
      - /mnt/tank/configs/penpot/penpot_assets:/opt/data/assets

    depends_on:
      - penpot-backend
      - penpot-admin-console
      - penpot-mcp
      - penpot-exporter

    environment:
      << : [*penpot-flags, *penpot-http-body-size, *penpot-public-uri]
      PENPOT_ADMIN_CONSOLE_URI: http://penpot-admin-console:3000

  penpot-backend:
    image: penpotapp/backend:2.18
    restart: unless-stopped

    volumes:
      - /mnt/tank/configs/penpot/penpot_assets:/opt/data/assets

    depends_on:
      penpot-postgres:
        condition: service_healthy
      penpot-valkey:
        condition: service_healthy

    environment:
      << : [*penpot-flags, *penpot-public-uri, *penpot-http-body-size, *penpot-secret-key, *penpot-database]

      PENPOT_ADMIN_CONSOLE_URI: http://penpot-admin-console:3000

      PENPOT_PREPL_HOST: 0.0.0.0

      PENPOT_REDIS_URI: redis://penpot-valkey/0

      PENPOT_OBJECTS_STORAGE_BACKEND: fs
      PENPOT_OBJECTS_STORAGE_FS_DIRECTORY: /opt/data/assets

      PENPOT_TELEMETRY_ENABLED: "false"
      PENPOT_TELEMETRY_REFERER: compose

      PENPOT_SMTP_DEFAULT_FROM: no-reply@example.com
      PENPOT_SMTP_DEFAULT_REPLY_TO: no-reply@example.com
      PENPOT_SMTP_HOST: penpot-mailpit
      PENPOT_SMTP_PORT: 1025
      PENPOT_SMTP_USERNAME:
      PENPOT_SMTP_PASSWORD:
      PENPOT_SMTP_TLS: "false"
      PENPOT_SMTP_SSL: "false"

  penpot-admin-console:
    image: penpotapp/admin-console:2.18
    restart: unless-stopped

    depends_on:
      penpot-postgres:
        condition: service_healthy

    environment:
      << : [*penpot-public-uri, *penpot-secret-key, *penpot-database]
      PENPOT_INTERNAL_URI: http://penpot-frontend:8080

  penpot-mcp:
    image: penpotapp/mcp:2.18
    restart: unless-stopped

  penpot-exporter:
    image: penpotapp/exporter:2.18
    restart: unless-stopped

    depends_on:
      penpot-valkey:
        condition: service_healthy

    environment:
      << : [*penpot-flags, *penpot-secret-key, *penpot-public-uri]
      PENPOT_INTERNAL_URI: http://penpot-frontend:8080
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

    environment:
      - VALKEY_EXTRA_FLAGS=--maxmemory 128mb --maxmemory-policy volatile-lfu

  penpot-mailpit:
    image: axllent/mailpit:v1
    restart: unless-stopped
    expose:
      - '1025'
    ports:
      - "8025:8025"
```

> Sent emails (sign-ups, invites) land in Mailpit at `http://IP:8025` until you configure a real SMTP provider.
{.is-info}