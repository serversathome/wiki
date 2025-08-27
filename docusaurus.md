---
title: Docusaurus
description: A guide to deploying Docusaurus in docker
published: true
date: 2025-08-27T12:17:08.556Z
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
  # Dev container (hot reload)
  docusaurus-dev:
    build:
      context: /mnt/tank/configs/docusaurus
      dockerfile: dockerfile
    working_dir: /app
    volumes:
      - /mnt/tank/configs/docusaurus:/app
      - /app/node_modules
    ports:
      - "9100:3000"
    command: yarn start --host 0.0.0.0
    environment:
      - NODE_ENV=development

  # Build container (one-off)
  docusaurus-build:
    build:
      context: /mnt/tank/configs/docusaurus
      dockerfile: dockerfile
    working_dir: /app
    volumes:
      - /mnt/tank/configs/docusaurus:/app
    command: yarn build

  # Prod container (serve static build)
  docusaurus-prod:
    build:
      context: /mnt/tank/configs/docusaurus
      dockerfile: dockerfile
    working_dir: /app
    volumes:
      - /mnt/tank/configs/docusaurus:/app
    ports:
      - "9200:5000"
    command: yarn serve
    depends_on:
      - docusaurus-build
    restart: unless-stopped

```

## 1.2 Dockerfile
> This `dockerfile` needs to be places in the `/mnt/tank/configs/docusaurus` dataset
{.is-info}


```yaml
FROM node:20-alpine

WORKDIR /app
RUN corepack enable

COPY package.json yarn.lock* ./
RUN if [ -f package.json ]; then yarn install; fi
COPY . .

EXPOSE 3000 5000

CMD ["yarn", "start"]

```

> The following commands assume your `stacks` directory for Dockge is at `/mnt/tank/stacks` you named this container `docusaurus` and the `configs` directory is at `/mnt/tank/configs`
{.is-warning}


1. Run the following command in the TrueNAS shell:
    ```bash
		docker run -it --name temp-docusaurus node:20-alpine npx create-docusaurus@latest /app classic && docker cp temp-docusaurus:/app/. /mnt/tank/configs/docusaurus/ && docker rm temp-docusaurus

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

## 2.1 Building a Production Site

To build the `prod` site after edits, run the following command in the TrueNAS shell:
```bash
docker compose -f /mnt/tank/stacks/docusaurus/compose.yaml run --rm docusaurus-build && \
docker compose -f /mnt/tank/stacks/docusaurus/compose.yaml up -d docusaurus-prod

```

When it asks `Do you want to continue? [Y/n]` hit <kbd>ENTER</kbd>.

Once the `prod` site is built, point your reverse proxy at `http://{IP}:9200` to serve your Docusaurus site. 


## 2.2 Editing the Files

