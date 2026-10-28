---
title: Dashy
description: A guide to deploying Dashy
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:14.446Z
---

# ![](/dashy.png){class="tab-icon"} What is Dashy?
A self-hostable personal dashboard built for you. Includes status-checking, widgets, themes, icon packs, a UI editor and tons more! 

<div class="glance">
  <div><span>Port</span><b><code>4000</code></b></div>
  <div><span>Deploy via</span><b>TrueNAS app or Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
</div>

# 1 · Deploy Dashy
# {.tabset}
## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Set the **Dashy Config Storage** to **Host Path**

## <img src="/docker.png" class="tab-icon"> Docker Compose
### Docker Compose
```yaml
services:
  dashy:
    container_name: Dashy
    image: lissy93/dashy
    user: "568:568"
    volumes:
      - /mnt/tank/configs/dashy:/app/user-data/
    ports:
      - 4000:8080
    environment:
      - NODE_ENV=production
    restart: unless-stopped
```

### Sample Config File
> Save the sample below as `/mnt/tank/configs/dashy/conf.yml` **before** starting the container. Dashy needs a `conf.yml` in the mounted folder, and an empty folder hides the default config.
{.is-warning}

```yaml

appConfig:
  theme: colorful
  layout: auto
  title: "My Dashboard"
  iconSize: medium
  headerStyle: boxed
  hideVersion: true

pageInfo:
  title: "My Dashboard"
  description: "Centralized access to all my self-hosted services"
  navLinks: []

sections:
  - name: "Section 1"
    icon: "fa-solid fa-folder"
    items: []
```