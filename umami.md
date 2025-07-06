---
title: Umami
description: A guide to installing Umami in docker via compose
published: true
date: 2025-07-06T10:30:45.970Z
tags: 
editor: markdown
dateCreated: 2024-06-30T21:47:45.888Z
---

![umami.png](/umami.png)

# What is Umami?

Umami is a simple, fast, privacy-focused, open-source analytics solution. Umami is a better alternative to [Google Analytics](https://marketingplatform.google.com/about/analytics/) because it gives you total control of your data and does not violate the privacy of your users.

# Docker Compose

```yaml
services:
 umami:
   image: ghcr.io/umami-software/umami:postgresql-latest
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

> To login, go to http://{IP}:3002. The default username is *admin* and the password is *umami*. 
{.is-info}


# Adding Websites

Navigate to **Settings** > **\+ Add Website**.

![](/screenshot_from_2024-06-30_17-48-16.png)

Name your website. For the domain, omit the http://. 

![](/screenshot_from_2024-06-30_17-49-09.png)

Navigate to **Websites** > then click the **Edit** button next to the site you just added. You will need the *Website ID* to link it to your website, and the *Tracking code* from the next tab.

![](/screenshot_from_2024-06-30_17-51-04.png)