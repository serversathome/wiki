---
title: Container Census
description: A guide to deploy Container Census
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:03:56.053Z
---

# What is Container Census?

Container Census is a lightweight, Go-powered tool that automatically scans your Docker environment across one or many hosts and gives you a clear, historical view of everything running in your stack.

<div class="glance">
  <div><span>Port</span><b><code>8080</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/selfhosters-cc/container-census"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

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
    environment:
      AUTH_ENABLED: false
      AUTH_USERNAME: your_username
      AUTH_PASSWORD: your_secure_password
      SESSION_SECRET: change-me # Required if AUTH_ENABLED is true. Generate with: openssl rand -hex 32
      TZ: ${TZ:-UTC}

    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:8080/api/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 10s

```

> All settings (scanner interval, vulnerability scanning, telemetry) are managed in the Web UI. Older versions used a `/app/config` folder; it is no longer needed and is only read once to migrate old settings.
{.is-info}

# <img src="/docker.png" class="tab-icon"> 2 · Agents for Other Hosts

To scan containers on other Docker hosts, run the Census agent on each one, then add it in the server UI with **+ Add Agent Host** (agent URL `http://<host-ip>:9876` plus the token).

```yaml
services:
  census-agent:
    image: ghcr.io/selfhosters-cc/census-agent:latest
    container_name: census-agent
    restart: unless-stopped
    group_add:
      - "${DOCKER_GID:-999}"
    ports:
      - "9876:9876"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      API_TOKEN: change-me # Generate with: openssl rand -hex 32
      PORT: 9876
      TZ: ${TZ:-UTC}
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:9876/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 5s
```