---
title: Mediqux
description: A guide to deploy Mediqux
published: true
date: 2025-09-29T14:54:20.239Z
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
      POSTGRES_DB: mediqux
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: admin
    volumes:
      - ./postgres_data:/var/lib/postgresql/data
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin -d mediqux"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    image: ghcr.io/dmjoh/mediqux/mediqux-backend:latest
    container_name: mediqux_backend
    environment:
      NODE_ENV: production
      DB_HOST: postgres
      DB_PORT: 5432
      DB_NAME: mediqux
      DB_USER: admin
      DB_PASSWORD: admin
      JWT_SECRET: choose-a-new-secret
      FRONTEND_URL: http://10.99.0.191:8086
      MAX_FILE_SIZE: 10MB
      UPLOAD_PATH: -/app/uploads
      PUID: 568
      PGID: 568
    ports:
      - "3000:3000"
    volumes:
      - ./mediqux_uploads:/app/uploads
    depends_on:
      postgres:
        condition: service_healthy
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "node", "-e", "require('http').get('http://localhost:3000/api/health', (res) => process.exit(res.statusCode === 200 ? 0 : 1))"]
      interval: 30s
      timeout: 10s
      retries: 3

  frontend:
    image: ghcr.io/dmjoh/mediqux/mediqux-frontend:latest
    container_name: mediqux_frontend
    environment:
      MEDIQUX_API_URL: http://10.99.0.191:8086
    ports:
      - "8086:80"
    depends_on:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80"]
      interval: 30s
      timeout: 10s
      retries: 3
```