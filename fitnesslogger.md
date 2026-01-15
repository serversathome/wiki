---
title: Fitness Logger
description: A guide to deploying Fitness Logger
published: true
date: 2026-01-15T15:29:14.990Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:41.398Z
---

# 💪 What is Fitness Logger?

Fitness Logger is a container I made to simply log workouts and show some progression reports based on exercise. Have suggestions, leave them below!


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Fitness Logger
```yaml
services:
  db:
    image: postgres:15-alpine
    container_name: fitness-db
    environment:
      POSTGRES_DB: fitnessdb
      POSTGRES_USER: fitnessuser
      POSTGRES_PASSWORD: fitnesspass
    volumes:
      - /mnt/tank/configs/fitnesslogger:/var/lib/postgresql/data
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U fitnessuser"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    image: serversathome/fitness-backend:latest
    container_name: fitness-backend
    environment:
      DATABASE_URL: postgresql://fitnessuser:fitnesspass@db:5432/fitnessdb
      SECRET_KEY: change-this-in-production
    ports:
      - "5000:5000"
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped
    environment:
  		- REGISTRATION_ENABLED=true

  frontend:
    image: serversathome/fitness-frontend:latest
    container_name: fitness-frontend
    ports:
      - "8085:80"
    depends_on:
      - backend
    restart: unless-stopped
```

# 2 · Logging In
1. Navigate to http://{IP}:8085
1. The default user is `admin` and the default password is `admin123`
