---
title: n8n
description: A guide to deploying n8n on TrueNAS as well as docker compose
published: true
date: 2025-07-24T16:54:38.031Z
tags: 
editor: markdown
dateCreated: 2025-07-24T16:54:38.031Z
---

# ![](/n8n.png){class="tab-icon"} What is n8n?

Uptime Kuma is a web monitor tool that supports various monitors such as HTTP, DNS, MySQL, Postgres, Docker, and more.

# 1 · Deploy n8n
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: kuma
    volumes:
      - /mnt/tank/configs/kuma:/app/data
    ports:
      - 3001:3001
    restart: unless-stopped
```

## <img src="/truenas.png" class="tab-icon"> TrueNAS

- The **App Config Storage** *Type of Storage* should be set to **Host Path** as described in [Folder Structure](/Folder-Structure).

# 2 · n8n Configuration

