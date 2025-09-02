---
title: Docusaurus
description: A guide to deploying Docusaurus in docker
published: true
date: 2025-09-02T22:36:11.546Z
tags: 
editor: markdown
dateCreated: 2025-08-27T08:38:39.465Z
---

# <img src="/docusaurus.png" class="tab-icon"> What is Docusaurus?
Docusaurus is an open-source static site generator designed for building documentation websites quickly and easily, primarily using Markdown. It allows users to create customizable sites while focusing on content creation, making it popular among software documentation teams.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Docusaurus

> Do not hit `Deploy` in Dockge. Hit the `Save` button then run the commands below from the shell!
{.is-danger}

## 1.1 Docker Compose
```yaml
services:

  dev:
    container_name: docusaurus-dev
    image: trevorbrunette/docusaurus:dev
    ports:
      - 3000:3000
    volumes:
      - ${APPS}/docusaurus:/opt/docusaurus
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
      - ${APPS}/docusaurus:/opt/docusaurus
    healthcheck:
      test:
        - CMD
        - bash
        - -c
        - "[ -f /opt/docusaurus/index.html ]"
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 3s

  server:
    depends_on:
      - build
    container_name: docusaurus-serve
    image: trevorbrunette/docusaurus:serve
    ports:
      - 8082:8080
    volumes:
      - ${APPS}/docusaurus:/opt/docusaurus
    healthcheck:
      test:
        - CMD
        - curl
        - -f
        - http://localhost:8080/
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 30s
networks: {}
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

