---
title: Umami
description: A guide to installing Umami in docker via compose
published: true
date: 2025-11-15T11:58:40.956Z
tags: 
editor: markdown
dateCreated: 2024-06-30T21:47:45.888Z
---

# <img src="/umami.png" class="tab-icon"> What is Umami?

Umami is a simple, fast, privacy-focused, open-source analytics solution. Umami is a better alternative to [Google Analytics](https://marketingplatform.google.com/about/analytics/) because it gives you total control of your data and does not violate the privacy of your users.

# 1 · Deploy Umami
# {.tabset}
## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Set a **Database Password**
1. Set an **App Secret**
1. Set the **WebUI Port** to `3002`
1. Set the **Postgres Data Storage** to **Host Path** and select the box for **Automatic Permissions**


## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
 umami:
   image: ghcr.io/umami-software/umami:latest
   container_name: umami
   ports:
     - "3002:3000"
   environment:
     DATABASE_URL: postgresql://umami:umami@db:5432/umami
     DATABASE_TYPE: postgresql
     APP_SECRET: replace-me-with-a-random-string
   depends_on:
     db:
       condition: service_healthy
   restart: unless-stopped
   healthcheck:
     test: ["CMD-SHELL", "curl http://localhost:3000/api/heartbeat"]
     interval: 5s
     timeout: 5s
     retries: 5
 db:
   image: postgres:15-alpine
   environment:
     POSTGRES_DB: umami
     POSTGRES_USER: umami
     POSTGRES_PASSWORD: umami
   volumes:
     - /mnt/tank/configs/umami:/var/lib/postgresql/data
   restart: unless-stopped
   healthcheck:
     test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
     interval: 5s
     timeout: 5s
     retries: 5
```

1. Change the default user names and passwords from the example. 

# 2 · Logging In

1. Navigate to `http://{IP}:3002`. The default username is *admin* and the password is *umami*.

# 3 · Adding Websites

Navigate to **Settings** > **\+ Add Website**.

![](/screenshot_from_2024-06-30_17-48-16.png)

Name your website. For the domain, omit the `http://`. 

![](/screenshot_from_2024-06-30_17-49-09.png)

Navigate to **Websites** > then click the **Edit** button next to the site you just added. You will need the *Website ID* to link it to your website, and the *Tracking code* from the next tab.

![](/screenshot_from_2024-06-30_17-51-04.png)