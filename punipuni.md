---
title: PuniPuni
description: A guide to deploying PuniPuni
published: true
date: '2026-11-04T16:23:28.594Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:07:40.075Z'
---

# What is PuniPuni?
PuniPuni is an user invitation system for Jellyfin, inspired by Wizarr. Built with Spring Boot and Angular, it allows user management for Jellyfin administrators through an interface and automation features.

# :logos:docker-icon: 1 · Deploy PuniPuni
Generate a JWT secret seed with `openssl rand -hex 32`, and replace the `change-me` passwords with your own.

```yaml
services:
  punipuni:
    image: kowlown/punipuni:latest
    restart: unless-stopped
    ports:
      - "8089:8080"
    environment:
      - PUNI_ADMIN_EMAIL=admin@example.com
      - PUNI_ADMIN_PASSWORD=change-me
      - PUNI_APPLICATION_DATABASE_TYPE=postgresql
      - PUNI_APPLICATION_DATABASE_HOST=postgres
      - PUNI_APPLICATION_DATABASE_PORT=5432
      - PUNI_APPLICATION_DATABASE_NAME=puni
      - PUNI_APPLICATION_DATABASE_USERNAME=puni
      - PUNI_APPLICATION_DATABASE_PASSWORD=change-me
      - PUNI_JWT_JWT_SECRET_SEED=your_generated_secret
    volumes:
      - /mnt/tank/configs/punipuni/data:/var/puni/data
    depends_on:
      - postgres

  postgres:
    image: postgres:17
    restart: unless-stopped
    environment:
      - POSTGRES_DB=puni
      - POSTGRES_USER=puni
      - POSTGRES_PASSWORD=change-me
    volumes:
      - /mnt/tank/configs/punipuni/postgres:/var/lib/postgresql/data
```

> [!NOTE]
> Log in at `http://IP:8089` with the `PUNI_ADMIN_EMAIL` and `PUNI_ADMIN_PASSWORD` you set. Older versions of this guide kept the database in `/mnt/tank/configs/punipuni/` itself, not `postgres/`.


