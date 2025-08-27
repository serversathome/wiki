---
title: Docusaurus
description: A guide to deploying Docusaurus in docker
published: true
date: 2025-08-27T08:38:39.465Z
tags: 
editor: markdown
dateCreated: 2025-08-27T08:38:39.465Z
---

# <img src="/docusaurus.png" class="tab-icon"> What is Docusaurus?
Docusaurus is an open-source static site generator designed for building documentation websites quickly and easily, primarily using Markdown. It allows users to create customizable sites while focusing on content creation, making it popular among software documentation teams.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Docusaurus
```yaml
services:
  # One-time scaffolder
  docusaurus-init:
    image: node:20-alpine
    working_dir: /work
    volumes:
      - ./:/work
    command: >
      sh -lc '
        set -e;
        corepack enable;
        # ensure ./site does not pre-exist to avoid CLI refusal
        if [ -e site ]; then
          if [ "$(ls -A site | wc -l)" -gt 0 ]; then
            echo "ERROR: ./site already exists and is not empty."; exit 1;
          else
            rmdir site;
          fi
        fi;
        npx create-docusaurus@latest site classic
      '
    profiles: ["init"]

  # Main dev server
  docusaurus:
    image: node:20-alpine
    container_name: docusaurus
    working_dir: /usr/src/app
    volumes:
      - ./site:/usr/src/app
    ports:
      - "9100:3000"
    environment:
      - HOST=0.0.0.0
      - CHOKIDAR_USEPOLLING=1   # helps file change detection on some hosts
    command: >
      sh -lc '
        set -e;
        corepack enable;
        if [ ! -f package.json ]; then
          echo "No Docusaurus project in ./site. Run: docker compose run --rm docusaurus-init";
          sleep 3600;  # stay up without restart loop
        else
          yarn install;
          yarn start --host 0.0.0.0;
        fi
      '
    restart: unless-stopped

```

# 2 · Configuration
