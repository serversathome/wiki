---
title: Kan
description: A guide to deploying Kan.bn in docker
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:05:45.849Z
---

# ![](/kan.png){class="tab-icon"} What is Kan?

A powerful, flexible kanban app that helps you organise work, track progress, and deliver results—all in one place.

<div class="glance">
  <div><span>Port</span><b><code>3000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>3 services</b></div>
  <div><span>Depends on</span><b>Postgres</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/kanbn/kan"><i class="mdi mdi-github"></i>Project</a>
    <a href="https://docs.kan.bn/introduction"><i class="mdi mdi-book-open-variant"></i>Docs</a>
    <a href="https://youtu.be/_Upe8mr5KMA"><i class="mdi mdi-youtube"></i>Video walkthrough</a>
  </div>
</div>

# Installation
```yaml
services:
  migrate:
    image: ghcr.io/kanbn/kan-migrate:latest
    container_name: kan-migrate
    environment:
      POSTGRES_URL: postgresql://kan:your_postgres_password@postgres:5432/kan_db
    depends_on:
      postgres:
        condition: service_healthy
    restart: "no"
  web:
    image: ghcr.io/kanbn/kan:latest
    container_name: kan-web
    ports:
      - 3000:3000
    environment:
      NEXT_PUBLIC_BASE_URL: http://10.99.0.191:3000
      BETTER_AUTH_SECRET: your_auth_secret
      POSTGRES_URL: postgresql://kan:your_postgres_password@postgres:5432/kan_db
      NEXT_PUBLIC_ALLOW_CREDENTIALS: true
    depends_on:
      migrate:
        condition: service_completed_successfully
    restart: unless-stopped
  postgres:
    image: postgres:15
    container_name: kan-db
    environment:
      POSTGRES_DB: kan_db
      POSTGRES_USER: kan
      POSTGRES_PASSWORD: your_postgres_password
    ports:
      - 5432:5432
    volumes:
      - /mnt/tank/configs/kanbn:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U kan -d kan_db"]
      interval: 5s
      timeout: 5s
      retries: 10
    restart: unless-stopped
```
- Change your `NEXT_PUBLIC_BASE_URL` to the correct value
- Replace `your_auth_secret` with a random string. Generate one with `openssl rand -hex 32`
- Use the same `your_postgres_password` value in all three places
- The `migrate` container runs the database migrations and then exits. That's expected: `web` starts once it finishes successfully.
> 
> Read the [official docs](https://docs.kan.bn/introduction)!
{.is-success}

# Video
https://youtu.be/_Upe8mr5KMA
