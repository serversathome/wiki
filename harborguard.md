---
title: Harbor Guard
description: A guide to deploying Harbor Guard
published: true
date: 2025-09-02T14:03:26.389Z
tags: 
editor: markdown
dateCreated: 2025-09-02T14:00:23.191Z
---

# ![](/harbor-guard.png){class="tab-icon"} What is Harbor Guard?

A comprehensive container security scanning platform that provides an intuitive web interface for managing and visualizing security assessments of Docker images.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Harbor Guard

```yaml
services:
  image: ghcr.io/harborguard/harborguard:latest
  restart: unless-stopped
  harborguard:
    ports:
      - 2998:8080
    environment:
      - PORT=8080
      - MAX_CONCURRENT_SCANS=5
      - LOG_LEVEL=info
      - HEALTH_CHECK_ENABLED=true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    
```
