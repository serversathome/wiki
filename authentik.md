---
title: Authentik
description: A guide to deploying Authentik
published: true
date: 2025-09-24T00:29:17.218Z
tags: 
editor: markdown
dateCreated: 2025-09-23T11:23:01.113Z
---

# ![](/authentik.png){class="tab-icon"} What is Authentik?

Authentik is an IdP (Identity Provider) and SSO (Single Sign On) platform that is built with security at the forefront of every piece of code, every feature, with an emphasis on flexibility and versatility.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Authentik
```yaml
services:
  postgresql:
    environment:
      POSTGRES_DB: authentik
      POSTGRES_PASSWORD: admin
      POSTGRES_USER: admin
    healthcheck:
      interval: 30s
      retries: 5
      start_period: 20s
      test:
        - CMD-SHELL
        - pg_isready -d $authentik -U $admin
      timeout: 5s
    image: docker.io/library/postgres:16-alpine
    restart: unless-stopped
    volumes:
      - /mnt/tank/authentik/database:/var/lib/postgresql/data # Leave this volume default
  
  redis:
    command: --save 60 1 --loglevel warning
    healthcheck:
      interval: 30s
      retries: 5
      start_period: 20s
      test:
        - CMD-SHELL
        - redis-cli ping | grep PONG
      timeout: 3s
    image: docker.io/library/redis:alpine
    restart: unless-stopped
    volumes:
      - /mnt/tank/authentik/redis:/data # Leave this volume default
  
  server:
    container_name: authentik
    command: server
    depends_on:
      postgresql:
        condition: service_healthy
      redis:
        condition: service_healthy
    user: root
    environment:
      AUTHENTIK_POSTGRESQL__HOST: postgresql
      AUTHENTIK_POSTGRESQL__NAME: authentik
      AUTHENTIK_POSTGRESQL__PASSWORD: admin
      AUTHENTIK_POSTGRESQL__USER: admin
      AUTHENTIK_REDIS__HOST: redis
      AUTHENTIK_SECRET_KEY: QJNn5SO37k99gl9NwDxvFfSKs/K5Brb1hhz/TiHJ/zMIn7JjC0ZjDmVcIlqdm0qukVp+Fj9tKfD/sdx4
    image: ghcr.io/goauthentik/server:2025.8.1
    ports:
      - 9000:9000 #http port
      - 9443:9443 #https port
    restart: unless-stopped
    volumes:
      - /mnt/tank/authentik/media:/media
      - /mnt/tank/authentik/templates:/templates

  worker:
    container_name: authentik-worker
    command: worker
    depends_on:
      postgresql:
        condition: service_healthy
      redis:
        condition: service_healthy
    environment:
      AUTHENTIK_POSTGRESQL__HOST: postgresql
      AUTHENTIK_POSTGRESQL__NAME: authentik
      AUTHENTIK_POSTGRESQL__PASSWORD: admin
      AUTHENTIK_POSTGRESQL__USER: admin
      AUTHENTIK_REDIS__HOST: redis
      AUTHENTIK_SECRET_KEY: QJNn5SO37k99gl9NwDxvFfSKs/K5Brb1hhz/TiHJ/zMIn7JjC0ZjDmVcIlqdm0qukVp+Fj9tKfD/sdx4
    image: ghcr.io/goauthentik/server:2025.8.1
    restart: unless-stopped
    user: root
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /mnt/tank/authentik/media:/media
      - /mnt/tank/authentik/certs:/certs
      - /mnt/tank/authentik/templates:/templates
```

1. Make a dataset in TrueNAS called `authentik` with 770 permmissions
1. To generate your own `POSTGRES_PASSWORD` run this command in the terminal:
    ```bash
    openssl rand -base64 36
    ```
1. To generate your own `AUTHENTIK_SECRET_KEY` run this command in the terminal:
    ```bash
    openssl rand -base64 60
    ```


