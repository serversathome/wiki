---
title: Uptime Kotlin
description: A guide to deploying Uptime Kotlin
published: true
date: 2026-10-31T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-10-31T12:00:00.000Z
---

# <img src="/uptime-kotlin.png" class="tab-icon"> What is Uptime Kotlin?

**Uptime Kotlin** is a lightweight uptime monitor with public status pages. It is compiled to a native image, so the whole app runs in roughly 50 MB of RAM, and everything it needs is stored in a single Postgres database.

- Monitors for HTTP/HTTPS, TCP, DNS, ping, SMTP, Postgres, MySQL/MariaDB, MSSQL, Redis, Kafka and RabbitMQ
- Real-time dashboard with response times, uptime history, automatic incidents and TLS certificate expiry tracking
- Notifications through Email, Discord, Slack, Teams, Telegram, Matrix, ntfy, Gotify, PagerDuty, webhooks and browser push
- Multiple public status pages and scheduled maintenance windows that mute alerts
- Prometheus metrics, a ready-made Grafana dashboard and optional worker containers for large setups

<div class="glance">
  <div><span>Port</span><b><code>8080</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>2 services</b></div>
  <div><span>Depends on</span><b>Postgres</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/TheoMeunier/uptime-kotlin"><i class="mdi mdi-github"></i>Project</a>
    <a href="https://uptime-kotlin.com/docs"><i class="mdi mdi-book-open-variant"></i>Docs</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Uptime Kotlin
```yaml
services:
  uptime-kotlin:
    image: ghcr.io/theomeunier/uptime-kotlin:latest
    container_name: uptime-kotlin
    restart: unless-stopped
    mem_limit: 512m
    ports:
      - 8080:8080
    environment:
      - TZ=America/New_York
      - QUARKUS_DATASOURCE_USERNAME=uptime-kotlin
      - QUARKUS_DATASOURCE_PASSWORD=CHANGE_ME
      - QUARKUS_DATASOURCE_JDBC_URL=jdbc:postgresql://uptime-kotlin-db:5432/uptime-kotlin
      - ENCRYPTION_MASTER_KEY=CHANGE_ME # openssl rand -base64 32
      - MP_JWT_VERIFY_ISSUER=https://issuer.uptime-kotlin.com
    depends_on:
      - uptime-kotlin-db

  uptime-kotlin-db:
    image: postgres:18-alpine
    container_name: uptime-kotlin-db
    restart: unless-stopped
    environment:
      - POSTGRES_DB=uptime-kotlin
      - POSTGRES_USER=uptime-kotlin
      - POSTGRES_PASSWORD=CHANGE_ME
    volumes:
      - /mnt/tank/configs/uptime-kotlin/postgres:/var/lib/postgresql
```

1. Generate a master key in the TrueNAS shell and paste it into `ENCRYPTION_MASTER_KEY`:
    ```bash
    openssl rand -base64 32
    ```
1. Change both `CHANGE_ME` passwords. `QUARKUS_DATASOURCE_PASSWORD` must match `POSTGRES_PASSWORD`
1. Set `TZ` to your timezone
1. Deploy the stack and open `http://YOUR-SERVER-IP:8080`

> The master key encrypts every stored secret (notification tokens, database passwords for monitors and the login signing key). Save it in your password manager. If you lose it, those secrets cannot be recovered.
{.is-danger}

> Keep `mem_limit: 512m`. The app sizes its memory from the limit it sees, and without one it will happily use several hundred MB instead of about 50.
{.is-info}

> Port `8080` is a popular default. If something else already uses it, change the left side of the mapping, for example `8095:8080`.
{.is-warning}

# 2 · Configuration

## 2.1 Create the Admin Account
1. On the first visit Uptime Kotlin asks you to create the administrator. There are no default credentials
1. Enter your name, email and a password of at least 8 characters

## 2.2 Add Monitors
1. Click **New Monitor** and pick the check type (HTTP, TCP, ping, DNS and so on)
1. Enter the target and the check interval
1. Save, and the monitor starts reporting on the dashboard right away

## 2.3 Notifications and Status Pages
1. Add a notification channel (Discord, ntfy, Gotify, email and others) and attach it to your monitors
1. Optionally enable reminders so you keep getting pinged while a service stays down
1. Create a **Status Page**, pick which monitors to show and choose a grid or list layout

> Browser push notifications only work over HTTPS or on `localhost`. Put Uptime Kotlin behind your reverse proxy if you want them.
{.is-info}

# <img src="/youtube.png" class="tab-icon"> 3 · Video

*No video yet, check back soon!*
