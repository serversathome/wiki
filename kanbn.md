---
title: Kan.bn
description: A guide to deploying Kan.bn in docker
published: true
date: 2025-07-23T23:37:23.061Z
tags: 
editor: markdown
dateCreated: 2025-07-07T17:56:48.924Z
---

# ![](/kan.png){class="tab-icon"} What is Kan?

A powerful, flexible kanban app that helps you organise work, track progress, and deliver results—all in one place.

# Installation
```yaml
services:
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
      - postgres
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
    restart: unless-stopped
```
- Change your `NEXT_PUBLIC_BASE_URL` to the correct value

