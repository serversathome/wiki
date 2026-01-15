---
title: Container Census
description: A guide to deploy Container Census
published: true
date: 2026-01-15T15:28:38.176Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:03:56.053Z
---

# What is Container Census?

Container Census is a lightweight, Go-powered tool that automatically scans your Docker environment across one or many hosts and gives you a clear, historical view of everything running in your stack.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Container Census

```yaml
services:
  census-server:
    image: ghcr.io/selfhosters-cc/container-census:latest
    container_name: census-server
    restart: unless-stopped
    group_add:
      - "${DOCKER_GID:-999}"

    ports:
      - "8080:8080"

    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /mnt/tank/configs/census/server:/app/data
      - /mnt/tank/configs/census/config:/app/config
    environment:
      AUTH_ENABLED: false
      AUTH_USERNAME: your_username
      AUTH_PASSWORD: your_secure_password
      TZ: ${TZ:-UTC}

    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:8080/api/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 10s

```