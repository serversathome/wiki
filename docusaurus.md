---
title: Docusaurus
description: A guide to deploying Docusaurus in docker
published: true
date: 2025-08-27T09:23:30.163Z
tags: 
editor: markdown
dateCreated: 2025-08-27T08:38:39.465Z
---

# <img src="/docusaurus.png" class="tab-icon"> What is Docusaurus?
Docusaurus is an open-source static site generator designed for building documentation websites quickly and easily, primarily using Markdown. It allows users to create customizable sites while focusing on content creation, making it popular among software documentation teams.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Docusaurus
```yaml
services:
  # One-time scaffolder (used only on first run)
  docusaurus-init:
    image: node:20-alpine
    working_dir: /work
    volumes:
      - /mnt/tank/configs/docusaurus:/work
    command: >
      sh -lc '
        set -e;
        corepack enable;
        npx create-docusaurus@latest . classic
      '
    profiles: ["init"]

  # Development server (hot reload)
  docusaurus-dev:
    image: node:20-alpine
    container_name: docusaurus-dev
    working_dir: /usr/src/app
    volumes:
      - /mnt/tank/configs/docusaurus:/usr/src/app
    ports:
      - "9100:3000"
    environment:
      - HOST=0.0.0.0
      - CHOKIDAR_USEPOLLING=1
    command: >
      sh -lc '
        set -e;
        corepack enable;
        if [ ! -f package.json ]; then
          echo "No Docusaurus project in /usr/src/app. Run: docker compose run --rm docusaurus-init";
          sleep 3600;
        else
          yarn install;
          yarn start --host 0.0.0.0;
        fi
      '
    restart: unless-stopped

  # Production server (static build served via nginx)
  docusaurus-prod:
    image: nginx:alpine
    container_name: docusaurus-prod
    volumes:
      - /mnt/tank/configs/docusaurus/build:/usr/share/nginx/html:ro
    ports:
      - "9200:80"
    restart: unless-stopped

  # Build step (generates /mnt/tank/configs/docusaurus/build)
  docusaurus-build:
    image: node:20-alpine
    working_dir: /usr/src/app
    volumes:
      - /mnt/tank/configs/docusaurus:/usr/src/app
    command: >
      sh -lc '
        set -e;
        corepack enable;
        yarn install;
        yarn build;
      '
    profiles: ["build"]
```
> The following commands assume your `stacks` directory for Dockge is at `/mnt/tank/stacks` and you named this container `docusaurus`
{.is-warning}


1. Run the following command in the TrueNAS shell:
    ```bash
    docker compose -f /mnt/tank/stacks/docusaurus/compose.yaml -p docusaurus run --rm docusaurus-init
    ```
	a.  When it asks `Ok to proceed? (y)` hit <kbd>ENTER</kbd>
  b. When it asks `Which language do you want to use?`  hit <kbd>ENTER</kbd>
1. Run the following command in the TrueNAS shell:
    ```bash
    docker compose -f /mnt/tank/stacks/docusaurus/compose.yaml up -d docusaurus-dev
    ```

# 2 · How it Works

Docusaurs has both a `dev` site running on port `9100` and a `production` site which will run on port `9200`. It is generally unsafe to expose the `dev` server to the internet, so Docusaurus "builds" a production site from the dev site for the public.

Everytime you make a change to the files in `/mnt/tank/configs/docusaurus` the `dev` site will be updated immediately. However, the `prod` site will need to be rebuilt every time. 

To build the `prod` site after edits, run the following command in the TrueNAS shell:
```bash
docker compose -f /mnt/tank/stacks/docusaurus/compose.yaml run --rm --profile build docusaurus-build && \
docker compose -f /mnt/tank/stacks/docusaurus/compose.yaml up -d docusaurus-prod
```

Once the `prod` site is built, point your reverse proxy at `http://{IP}:9200` to serve your Docusaurus site. 