---
title: Phylum
description: A guide to deploying Phylum via docker
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:10.430Z
---

# ![](/phylum.png){class="tab-icon"} What is Phylum?
Phylum is a self-hosted file storage platform with offline-first web and native clients, meant as a replacement for Google Drive, Dropbox, etc.

<div class="glance">
  <div><span>Port</span><b><code>2448</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>2 services</b></div>
  <div><span>Depends on</span><b>Postgres</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://codeberg.org/shroff/phylum"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Phylum
Generate a database password and use it for both `PHYLUM_DB__PASSWORD` and `POSTGRES_PASSWORD`:

```bash
openssl rand -hex 32
```

```yaml
services:
  server:
    container_name: phylum_server
    image: shroff12/phylum:latest
    volumes:
      - /mnt/tank/configs/phylum/data:/var/lib/phylum
    environment:
      PHYLUM_DB__HOST: db
      PHYLUM_DB__PORT: 5432
      PHYLUM_DB__NAME: phylum
      PHYLUM_DB__USER: phylum
      PHYLUM_DB__PASSWORD: your_db_password
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
      POSTGRES_PASSWORD: your_db_password
    volumes:
      - /mnt/tank/configs/phylum/postgres:/var/lib/postgresql/data
    expose:
      - 5432
    healthcheck:
      test: "pg_isready -U phylum -d phylum"
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 5s
    restart: unless-stopped

```

> Phylum does not run as the `apps` user so be sure to create generic permission datasets and give other read, write, and execute access!
{.is-warning}


1. Open `http://IP:2448` (or connect with one of the native apps). On first run Phylum starts in setup mode, so follow the setup flow to configure the instance and create your first user.
> 
> The password must be at least 8 characters long
{.is-warning}

> **Upgrading from an older install?** Earlier versions of this page mounted `/mnt/tank/configs/phylum/storage:/app/storage`. v0.9.0 moved Phylum's data to `/var/lib/phylum` (with `config.yml` and `storage` inside it), and v0.10.0 renamed the config env vars (each `.` is now `__`, e.g. `PHYLUM_DB__HOST`) and changed the storage layout. Stop the stack, then move your old files into the new folder **before** you start it with the new compose:
> ```bash
> mkdir -p /mnt/tank/configs/phylum/data
> mv /mnt/tank/configs/phylum/storage /mnt/tank/configs/phylum/data/storage
> ```
> Move `config.yml` into `/mnt/tank/configs/phylum/data/` as well if you have one. Then update your compose as above and migrate your data with `docker compose run --rm server admin migrate img previews` and `docker compose run --rm server admin migrate localstorage`. Read the [v0.9.0](https://codeberg.org/shroff/phylum/releases/tag/releases/v0.9.0) and [v0.10.0](https://codeberg.org/shroff/phylum/releases/tag/releases/v0.10.0) release notes before upgrading.
{.is-warning}


> [Read the official documentation!](https://codeberg.org/shroff/phylum)
{.is-success}
