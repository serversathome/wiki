---
title: PuniPuni
description: A guide to deploying PuniPuni
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:40.075Z
---

# <img src="/punipuni.png" class="tab-icon"> What is PuniPuni?
PuniPuni is an user invitation system for Jellyfin, inspired by Wizarr. Built with Spring Boot and Angular, it allows user management for Jellyfin administrators through an interface and automation features.

<div class="glance">
  <div><span>Port</span><b><code>8089</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>2 services</b></div>
  <div><span>Depends on</span><b>Postgres</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy PuniPuni
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

> Log in at `http://IP:8089` with the `PUNI_ADMIN_EMAIL` and `PUNI_ADMIN_PASSWORD` you set. Older versions of this guide kept the database in `/mnt/tank/configs/punipuni/` itself, not `postgres/`.
{.is-info}

