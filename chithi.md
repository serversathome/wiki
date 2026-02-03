---
title: Chithi
description: A guide to deploying Chithi
published: true
date: 2026-02-03T16:57:25.096Z
tags: 
editor: markdown
dateCreated: 2026-02-03T16:57:15.577Z
---

#  What is Chithi?

An end-to-end encrypted file sharing application with temporary links, user management, and background cleanup tasks.


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Chithi

> 
> Chithi does not have pre-built Docker images and must use docker build
{.is-info}

1. Run the following command in the `stacks` directory of your TrueNAS shell:
   ```bash
   git clone https://github.com/chithi-dev/chithi.git
   ```
2. Edit `docker-compose.yml`:
   - Change the default credentials: Postgres password (`supersecretpassword`) and RustFS keys (`rustfsadmin`)
   - Update the `DOMAIN` environment variable in the caddy service and `PUBLIC_BACKEND_API` in the frontend service to match your server
   - If you already have a reverse proxy, change the caddy service ports:
     ```yaml
     caddy:
       ports:
         - "8080:80"
         - "8443:443"
     ```
3. Navigate to your container manager and launch the stack with build enabled

# 2 · First Login

1. Access the web UI via your reverse proxy, or directly at `http://your-server:8080`
2. Create your account through the onboarding flow

# <img src="/youtube.png" class="tab-icon"> 3 · Video

*No video yet — check back soon!*