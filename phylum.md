---
title: Phylum
description: A guide to deploying Phylum via docker
published: true
date: 2025-08-19T00:19:57.902Z
tags: 
editor: markdown
dateCreated: 2025-08-04T12:25:07.864Z
---

# ![](/phylum.png){class="tab-icon"} What is Phylum?
Phylum is a self-hosted file storage platform with offline-first web and native clients, meant as a replacement for Google Drive, Dropbox, etc.
# <img src="/docker.png" class="tab-icon"> 1 · Deploy Phylum
```yaml
services:
  server:
    container_name: phylum_server
    image: shroff12/phylum:latest
    volumes:
      - /mnt/tank/configs/phylum/storage:/app/storage
      - /mnt/tank/configs/phylum/storage/config.yml:/app/config.yml
    environment:
      PHYLUM_DB_HOST: db
    ports:
      - '2448:2448'
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped

  db:
    container_name: phylum_db
    image: docker.io/postgres:17
    environment:
      POSTGRES_DB: phylum
      POSTGRES_USER: phylum
      POSTGRES_PASSWORD: changeme
    volumes:
      - /mnt/tank/configs/phylum/postgres:/var/lib/postgresql/data
    ports:
      - '5432:5432'
    healthcheck:
      test: "pg_isready -U phylum -d phylum"
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 5s
    restart: unless-stopped

```
Run the following command in the TrueNAS shell before launching the container:
```bash
rm -rf /mnt/tank/configs/phylum/storage/config.yml && touch /mnt/tank/configs/phylum/storage/config.yml
```

Then create a user by running the following command in the TrueNAS shell replacing the `email` with your user:
```bash
docker exec -it phylum_server phylum admin user create email --no-email
```

> [Read the official documentation!](https://codeberg.org/shroff/phylum)
{.is-success}
