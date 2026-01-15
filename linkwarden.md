---
title: Linkwarden
description: A guide to deploying Linkwarden
published: true
date: 2026-01-15T15:29:57.316Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:05:57.751Z
---

# ![](/linkwarden.png){class="tab-icon"} What is Linkwarden?

Linkwarden is a self-hosted, open-source collaborative bookmark manager to collect, read, annotate, and fully preserve what matters, all in one place.

The objective is to organize useful webpages and articles you find across the web in one place, and since useful webpages can go away (see the inevitability of Link Rot), Linkwarden also saves a copy of each webpage as a Screenshot and PDF, ensuring accessibility even if the original content is no longer available.

In addition to preservation, Linkwarden provides a user-friendly reading and annotation experience that blends the simplicity of a “read-it-later” tool with the reliability of a web archive. Whether you’re highlighting key ideas, jotting down thoughts, or revisiting content long after it’s disappeared from the web, Linkwarden keeps your knowledge accessible and organized.

Linkwarden is also designed with collaboration in mind, enabling you to share links with the public and/or collaborate seamlessly with multiple users.

# 1 · Deploy Linkwarden
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  postgres:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
    	- POSTGRES_PASSWORD=admin
    volumes:
      - /mnt/tank/configs/linkwarden/pgdata:/var/lib/postgresql/data
  
  linkwarden:
    environment:
      - DATABASE_URL=postgresql://postgres:admin@postgres:5432/postgres
      - NEXTAUTH_URL=http://localhost:3000/api/v1/auth
      - NEXTAUTH_SECRET=VERY_SENSITIVE_SECRET
    restart: unless-stopped
    image: ghcr.io/linkwarden/linkwarden:latest 
    ports:
      - 3000:3000
    volumes:
      - /mnt/tank/configs/linkwarden/data:/data/data
  
  meilisearch:
    image: getmeili/meilisearch:v1.12.8
    restart: unless-stopped
    environment:
    	- MEILI_MASTER_KEY=VERY_SENSITIVE_MEILI_MASTER_KEY
    volumes:
      - /mnt/tank/configs/linkwarden/meili_data:/meili_data
```


## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Set a **Database Password**, **NextAuth Secret**, **Meilisearch Master Key**
1. Set the **Storage Configuration** for **Data Storage**, **Meilisearch Data Storage**, and **Postgres Data Storage** to *Host Path*

# 2 · Logging In
1. Navigate to `http://PORT:IP` 
1. Click **Sign Up** at the bottom of the window
1. Enter your credentials then click **Sign Up**
1. Login with your created credentials

# <img src="/youtube.png" class="tab-icon"> 3 · Video
https://youtu.be/VV9Vuh_r1RY
