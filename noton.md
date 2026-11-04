---
title: Noton
description: A guide to deploying Noton
published: true
date: '2026-11-04T16:23:20.412Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:06:50.817Z'
---

# What is Noton?
A free-for-internal-use documentation platform (Functional Source License 1.1; competing commercial use is not allowed) built with Laravel and Filament, enhanced by Ollama for local AI features, focused on clarity, structure, and self-hosted simplicity. 

# :logos:docker-icon: 1 · Deploy Noton
```yaml
services:
  noton:
    container_name: noton
    image: ghcr.io/bartvantuijn/noton:latest
    restart: unless-stopped
    ports:
      - 6686:6686
    environment:
      APP_NAME: Noton
      APP_ENV: local
      APP_DEBUG: true
      APP_URL: http://10.99.0.191:6686
      APP_LOCALE: en
      DB_CONNECTION: pgsql
      DB_HOST: postgres
      DB_PORT: 5432
      DB_DATABASE: noton_database
      DB_USERNAME: noton_user
      DB_PASSWORD: noton_password
      OLLAMA_BASE_URL: http://ollama:11434
      OLLAMA_MODEL: llama3.1:8b
      OLLAMA_PULL_TIMEOUT: 600
    volumes:
      - /mnt/tank/configs/noton/uploads:/srv/www/storage/app/public
  postgres:
    container_name: postgres
    image: postgres:16
    restart: unless-stopped
    environment:
      POSTGRES_DB: noton_database
      POSTGRES_USER: noton_user
      POSTGRES_PASSWORD: noton_password
    volumes:
      - /mnt/tank/configs/noton/db:/var/lib/postgresql/data
  ollama:
    container_name: ollama
    image: ollama/ollama:latest
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/noton/ollama:/root/.ollama
```

> [!NOTE]
> Already run Ollama? Remove the `ollama` service and point `OLLAMA_BASE_URL` at it.

1. Navigate to `http://IP:6686`
1. Register the first user. This account is automatically assigned the admin role

