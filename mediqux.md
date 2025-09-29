---
title: Mediqux
description: A guide to deploy Mediqux
published: true
date: 2025-09-29T14:47:33.544Z
tags: 
editor: markdown
dateCreated: 2025-09-29T14:47:33.544Z
---

# 🏥 What is Mediqux?
A comprehensive medical record system for individuals and families. Built for complete local deployment with automated lab report processing.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Mediqux
```yaml
services:
  postgres:
    image: postgres:17-alpine
    container_name: mediqux_postgres
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - mediqux_network
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    image: ghcr.io/dmjoh/mediqux/mediqux-backend:${IMAGE_TAG}
    container_name: mediqux_backend
    environment:
      NODE_ENV: production
      DB_HOST: postgres
      DB_PORT: ${DB_PORT}
      DB_NAME: ${POSTGRES_DB}
      DB_USER: ${POSTGRES_USER}
      DB_PASSWORD: ${POSTGRES_PASSWORD}
      JWT_SECRET: ${JWT_SECRET}
      FRONTEND_URL: ${FRONTEND_URL}
      MAX_FILE_SIZE: ${MAX_FILE_SIZE:-10MB}
      UPLOAD_PATH: ${UPLOAD_PATH:-/app/uploads}
      PUID: ${PUID:-1000}
      PGID: ${PGID:-1000}
    ports:
      - "${BACKEND_PORT:-3000}:3000"
    volumes:
      - mediqux_uploads:/app/uploads
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - mediqux_network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "node", "-e", "require('http').get('http://localhost:3000/api/health', (res) => process.exit(res.statusCode === 200 ? 0 : 1))"]
      interval: 30s
      timeout: 10s
      retries: 3

  frontend:
    image: ghcr.io/dmjoh/mediqux/mediqux-frontend:${IMAGE_TAG}
    container_name: mediqux_frontend
    environment:
      MEDIQUX_API_URL: ${MEDIQUX_API_URL}
    ports:
      - "${FRONTEND_PORT:-8080}:80"
    depends_on:
      - backend
    networks:
      - mediqux_network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80"]
      interval: 30s
      timeout: 10s
      retries: 3

```