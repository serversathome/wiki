---
title: Harbor Guard
description: A guide to deploying Harbor Guard
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:05:01.209Z
---

# ![](/harbor-guard.png){class="tab-icon"} What is Harbor Guard?

A comprehensive container security scanning platform that provides an intuitive web interface for managing and visualizing security assessments of Docker images.

<div class="glance">
  <div><span>Port</span><b><code>2998</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/harborguard/harborguard"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Harbor Guard

```yaml
services:
  harborguard:
    image: ghcr.io/harborguard/harborguard:latest
    restart: unless-stopped
    ports:
      - 2998:8080
    environment:
      - PORT=8080
      - MAX_CONCURRENT_SCANS=5
      - LOG_LEVEL=info
      - HEALTH_CHECK_ENABLED=true
      # Optional: push notifications to Gotify (Slack, Teams and Apprise are also supported)
      # - GOTIFY_SERVER_URL=https://gotify.example.com
      # - GOTIFY_APP_TOKEN=your-app-token
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - harborguard-db:/var/lib/postgresql/data

volumes:
  harborguard-db:
```

> Harbor Guard keeps its scan history in a bundled PostgreSQL database at `/var/lib/postgresql/data`. Without a volume there, every scan is lost when the container is recreated. A named volume is used here rather than a `/mnt/tank/configs` bind mount because it inherits the `postgres` user's ownership from the image; a fresh host folder owned by root makes the database fail to initialise. To use your own PostgreSQL instead, set `DATABASE_URL=postgresql://user:pass@host:5432/harborguard`.
{.is-info}

Other useful options: `SCAN_TIMEOUT_MINUTES` (default `30`), `ENABLED_SCANNERS` (default `trivy,grype,syft,dockle,osv,dive`) and `CLEANUP_OLD_SCANS_DAYS` (default `30`). Auto-patching images requires running the container with `privileged: true`.
