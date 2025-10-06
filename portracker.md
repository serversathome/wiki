---
title: Portracker
description: A guide to deploying Portracker via docker
published: true
date: 2025-10-06T14:03:28.536Z
tags: 
editor: markdown
dateCreated: 2025-08-04T12:30:09.026Z
---

# ![](/portracker.png){class="tab-icon"} What is Portracker?
By auto-discovering services on your systems, portracker provides a live, accurate map of your network. It helps eliminate manual tracking in spreadsheets and prevents deployment failures caused by port conflicts.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Portracker

```yaml
services:
  portracker:
    image: mostafawahied/portracker:latest
    container_name: portracker
    restart: unless-stopped
    pid: "host"
    ports:
    	- 4999:4999
    cap_add:
      - SYS_PTRACE
      - SYS_ADMIN
    security_opt:
      - apparmor:unconfined
    volumes:
      - /mnt/tank/configs/portracker:/data
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      - DATABASE_PATH=/data/portracker.db
      - PORT=4999
      # Optional: For enhanced TrueNAS features
      # - TRUENAS_API_KEY=your-api-key-here
```

# 2 · Integrating With TrueNAS

To geenrate an API key for TrueNAS, click the username in the top right of the web UI and select **My API Keys**. Click **Add** in the top right and generate a new API key, then copy it into the compose file above.