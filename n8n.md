---
title: n8n
description: A guide to deploying n8n on TrueNAS as well as docker compose
published: true
date: 2025-07-24T17:10:51.469Z
tags: 
editor: markdown
dateCreated: 2025-07-24T16:54:38.031Z
---

# ![](/n8n.png){class="tab-icon"} What is n8n?

n8n offers a unique workflow automation platform combining AI and business process automation for technical teams, blending coding flexibility with no-code speed.

# 1 · Deploy n8n
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    ports:
      - 5678:5678
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=securepassword
      - TZ=America/New_York
      - N8N_SECURE_COOKIE=false
      - WEBHOOK_URL=
    volumes:
      - /mnt/bigdeal/configs/n8n:/home/node/.n8n
    restart: unless-stopped
    user: 568:568
```

## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Set the **Web Host** to the IP of the server or your FQDN
1. Set a **Database Password**
1. Set a **Redis Password**
1. Set an **Encryption Key**
1. The **Storage Configuration** *n8n Data Storage* should be set to **Host Path** as described in [Folder Structure](/Folder-Structure).
1. The **Storage Configuration** *n8n Postgres Data Storage* should be set to **Host Path** and the box for ** Automatic Permissions** needs to be checked
1. Increase **Resources Configuration**

# 2 · n8n Configuration

