---
title: Authentik
description: A guide to deploying Authentik
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:03:13.956Z
---

# ![](/authentik.png){class="tab-icon"} What is Authentik?

Authentik is an IdP (Identity Provider) and SSO (Single Sign On) platform that is built with security at the forefront of every piece of code, every feature, with an emphasis on flexibility and versatility.

<div class="glance">
  <div><span>Port</span><b><code>9000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>3 services</b></div>
  <div><span>Depends on</span><b>Postgres</b></div>
  <div><span>Difficulty</span><b class="difficulty intermediate">Intermediate</b></div>
  <div class="glance-links">
    <a href="https://github.com/goauthentik/authentik"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Authentik
```yaml
services:
  postgresql:
    environment:
      POSTGRES_DB: authentik
      POSTGRES_PASSWORD: CHANGEME_DB_PASSWORD
      POSTGRES_USER: admin
    healthcheck:
      interval: 30s
      retries: 5
      start_period: 20s
      test:
        - CMD-SHELL
        - pg_isready -d $${POSTGRES_DB} -U $${POSTGRES_USER}
      timeout: 5s
    image: docker.io/library/postgres:16-alpine
    restart: unless-stopped
    volumes:
      - /mnt/tank/authentik/database:/var/lib/postgresql/data # Leave this volume default

  server:
    container_name: authentik
    command: server
    depends_on:
      postgresql:
        condition: service_healthy
    user: root
    environment:
      AUTHENTIK_POSTGRESQL__HOST: postgresql
      AUTHENTIK_POSTGRESQL__NAME: authentik
      AUTHENTIK_POSTGRESQL__PASSWORD: CHANGEME_DB_PASSWORD
      AUTHENTIK_POSTGRESQL__USER: admin
      AUTHENTIK_SECRET_KEY: CHANGEME_SECRET_KEY
    image: ghcr.io/goauthentik/server:2026.8.5
    ports:
      - 9000:9000 #http port
      - 9443:9443 #https port
    restart: unless-stopped
    shm_size: 512mb
    volumes:
      - /mnt/tank/authentik/data:/data
      - /mnt/tank/authentik/templates:/templates

  worker:
    container_name: authentik-worker
    command: worker
    depends_on:
      postgresql:
        condition: service_healthy
    environment:
      AUTHENTIK_POSTGRESQL__HOST: postgresql
      AUTHENTIK_POSTGRESQL__NAME: authentik
      AUTHENTIK_POSTGRESQL__PASSWORD: CHANGEME_DB_PASSWORD
      AUTHENTIK_POSTGRESQL__USER: admin
      AUTHENTIK_SECRET_KEY: CHANGEME_SECRET_KEY
    image: ghcr.io/goauthentik/server:2026.8.5
    restart: unless-stopped
    shm_size: 512mb
    user: root
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /mnt/tank/authentik/data:/data
      - /mnt/tank/authentik/certs:/certs
      - /mnt/tank/authentik/templates:/templates
```

1. Make a dataset in TrueNAS called `authentik` with 770 permmissions
1. To generate your own database password run this command in the terminal, then replace all three `CHANGEME_DB_PASSWORD` values with it:
    ```bash
    openssl rand -base64 36
    ```
1. To generate your own `AUTHENTIK_SECRET_KEY` run this command in the terminal, then replace both `CHANGEME_SECRET_KEY` values with it:
    ```bash
    openssl rand -base64 60
    ```

> **Upgrading an existing install?** Authentik must be upgraded one major release at a time, so do not jump straight to 2026.8. From 2025.8, step through 2025.10, 2025.12, 2026.2 and 2026.5 first, letting each one start fully. Since 2025.10 Authentik no longer uses Redis, so the `redis` service and `AUTHENTIK_REDIS__HOST` can be removed. Since 2025.12 files are stored under `/data`: before starting 2025.12, move your old media folder with `mkdir -p /mnt/tank/authentik/data && mv /mnt/tank/authentik/media /mnt/tank/authentik/data/media`. See the [upgrade guide](https://docs.goauthentik.io/install-config/upgrade/) and the [2025.10](https://docs.goauthentik.io/releases/2025.10/) and [2025.12](https://docs.goauthentik.io/releases/2025.12/) release notes.
{.is-warning}


