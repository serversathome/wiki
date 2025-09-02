---
title: Docusaurus
description: A guide to deploying Docusaurus in docker
published: true
date: 2025-09-02T22:43:14.434Z
tags: 
editor: markdown
dateCreated: 2025-08-27T08:38:39.465Z
---

# <img src="/docusaurus.png" class="tab-icon"> What is Docusaurus?
Docusaurus is an open-source static site generator designed for building documentation websites quickly and easily, primarily using Markdown. It allows users to create customizable sites while focusing on content creation, making it popular among software documentation teams.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Docusaurus


```yaml
services:

  dev:
    container_name: docusaurus-dev
    image: trevorbrunette/docusaurus:dev
    ports:
      - 3000:3000
    volumes:
      - /mnt/tank/configs/docusaurus:/opt/docusaurus
    healthcheck:
      test:
        - CMD
        - curl
        - -f
        - http://localhost:3000/
      interval: 30s
      timeout: 10s
      retries: 3
      # Give More Time To Start Up If Failed 
      start_period: 120s

  build:
    depends_on:
      dev:
        condition: service_healthy
    container_name: docusaurus-build
    image: trevorbrunette/docusaurus:build
    restart: no
    volumes:
      - /mnt/tank/configs/docusaurus:/opt/docusaurus

  server:
    container_name: docusaurus-serve
    restart: unless-stopped
    image: trevorbrunette/docusaurus:serve
    ports:
      - 8082:8080
    volumes:
      - /mnt/tank/configs/docusaurus/docusaurus:/opt/docusaurus


  code-server:
    image: lscr.io/linuxserver/code-server:latest
    container_name: code-server
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
      - PASSWORD=password #optional
      - HASHED_PASSWORD= #optional
      - SUDO_PASSWORD=password #optional
      - SUDO_PASSWORD_HASH= #optional
      - PROXY_DOMAIN=code-server.my.domain #optional
      - DEFAULT_WORKSPACE=/mnt/tank/configs/docusaurus #optional
      - PWA_APPNAME=code-server #optional
    volumes:
      - /mnt/tank/configs/codeserver:/config
    ports:
      - 8443:8443
    restart: unless-stopped
```

# 2 · How it Works

Docusaurs has both a `dev` site running on port `9100` and a `production` site which will run on port `9200`. It is generally unsafe to expose the `dev` server to the internet, so Docusaurus "builds" a production site from the dev site for the public.

Everytime you make a change to the files in `/mnt/tank/configs/docusaurus` the `dev` site will be updated immediately. However, the `prod` site will need to be rebuilt every time. 

## 2.1 Building a Production Site

To build the `prod` site after edits, run the following command in the TrueNAS shell:
```bash

```


Once the `prod` site is built, point your reverse proxy at `http://{IP}:9200` to serve your Docusaurus site. 


## 2.2 Editing the Files

