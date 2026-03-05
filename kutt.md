---
title: Kutt
description: A guide to deploying Kutt
published: true
date: 2026-03-05T18:36:14.958Z
tags: 
editor: markdown
dateCreated: 2026-03-05T18:35:20.228Z
---

# <img src="/kutt.png" class="tab-icon"> What is Kutt?

**Kutt** is a free, open-source, self-hosted URL shortener with support for custom domains. It allows you to create and manage shortened links, set custom addresses, protect links with passwords, configure expiration times, view private click statistics, and integrate with other tools through a RESTful API. Kutt supports SQLite (default), PostgreSQL, and MySQL/MariaDB as database backends, and optionally Redis for caching.


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Kutt
# {.tabset}
## SQLite

```yaml
services:
  kutt:
    image: kutt/kutt:latest
    container_name: kutt
    restart: unless-stopped
    depends_on:
      redis:
        condition: service_started
    ports:
      - "3000:3000"
    volumes:
      - /mnt/tank/configs/kutt/data:/var/lib/kutt
      - /mnt/tank/configs/kutt/custom:/kutt/custom
    environment:
      - DB_FILENAME=/var/lib/kutt/data.sqlite
      - REDIS_ENABLED=true
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - JWT_SECRET=
      - DEFAULT_DOMAIN=
      - SITE_NAME=Kutt
      - DISALLOW_REGISTRATION=false
      - DISALLOW_ANONYMOUS_LINKS=true

  redis:
    image: redis:alpine
    container_name: kutt-redis
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/kutt/redis:/data
```


> You **must** change `JWT_SECRET` to a long random string before deploying. You can generate one with: `openssl rand -base64 32`
{.is-danger}

> Change `DEFAULT_DOMAIN` to the domain or IP address you will use to access Kutt. Do not include `http://` or `https://` — just the domain (e.g., `links.yourdomain.com`).
{.is-info}

## PostgreSQL + Redis

If you prefer PostgreSQL over SQLite, use this compose instead:

```yaml
services:
  kutt:
    image: kutt/kutt:latest
    container_name: kutt
    restart: unless-stopped
    depends_on:
      postgres:
        condition: service_started
      redis:
        condition: service_started
    command: ["./wait-for-it.sh", "postgres:5432", "--", "npm", "start"]
    ports:
      - "3000:3000"
    volumes:
      - /mnt/tank/configs/kutt/custom:/kutt/custom
    environment:
      - DB_HOST=postgres
      - DB_PORT=5432
      - DB_NAME=kutt
      - DB_USER=kutt
      - DB_PASSWORD=CHANGE_ME
      - REDIS_ENABLED=true
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - JWT_SECRET=
      - DEFAULT_DOMAIN=
      - SITE_NAME=Kutt
      - DISALLOW_REGISTRATION=false
      - DISALLOW_ANONYMOUS_LINKS=true

  redis:
    image: redis:alpine
    container_name: kutt-redis
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/kutt/redis:/data

  postgres:
    image: postgres:16-alpine
    container_name: kutt-postgres
    restart: unless-stopped
    environment:
      - POSTGRES_USER=kutt
      - POSTGRES_PASSWORD=CHANGE_ME
      - POSTGRES_DB=kutt
    volumes:
      - /mnt/tank/configs/kutt/postgres:/var/lib/postgresql/data
```
> You **must** change `JWT_SECRET` to a long random string before deploying. You can generate one with: `openssl rand -base64 32`
{.is-danger}

> Change `DEFAULT_DOMAIN` to the domain or IP address you will use to access Kutt. Do not include `http://` or `https://` — just the domain (e.g., `links.yourdomain.com`).
{.is-info}

> Make sure `DB_USER` / `DB_PASSWORD` / `DB_NAME` match the `POSTGRES_USER` / `POSTGRES_PASSWORD` / `POSTGRES_DB` values.
{.is-warning}

# 2 · Configuration

## 2.1 Initial Setup

When you first access Kutt after deployment, you will be prompted to create an admin account. Enter your email and password to get started.

Once logged in, you'll see a clean interface with a URL input box at the top. Paste any long URL and click the button to generate a short link.

## 2.2 Environment Variables

| Variable | Description | Default |
|---|---|---|
| `JWT_SECRET` | **Required.** Long random string for signing auth tokens. | — |
| `DEFAULT_DOMAIN` | Domain where Kutt is hosted (no http/https). | `localhost:3000` |
| `SITE_NAME` | Display name of your Kutt instance. | `Kutt` |
| `PORT` | Port Kutt runs on inside the container. | `3000` |
| `LINK_LENGTH` | Number of characters in generated short codes. | `6` |
| `DISALLOW_REGISTRATION` | Set to `true` to prevent new signups. | `false` |
| `DISALLOW_ANONYMOUS_LINKS` | Set to `true` to require login for creating links. | `false` |
| `USER_LIMIT_PER_DAY` | Max links a user can create per day. | `50` |
| `NON_USER_COOLDOWN` | Cooldown in minutes for anonymous users. `0` to disable. | `0` |
| `DEFAULT_MAX_STATS_PER_LINK` | Max number of visits stored per link for detailed stats. | `5000` |
| `CUSTOM_DOMAIN_USE_HTTPS` | Set to `true` if using a reverse proxy with SSL. | `false` |
| `ADMIN_EMAILS` | Comma-separated list of admin email addresses. | — |
| `REDIS_ENABLED` | Enable Redis caching. | `false` |
| `REDIS_HOST` | Redis hostname. | — |
| `REDIS_PORT` | Redis port. | `6379` |
{.dense}

> As of v3.2.3, you can set any environment variable by reading from a file. Append `_FILE` to the variable name and set the value to the file path. Example: `JWT_SECRET_FILE=/run/secrets/jwt_secret`
{.is-info}

## 2.3 Link Options

When creating a short link, you can optionally configure:

- **Custom Address** — Use a human-readable slug instead of a random code (e.g., `yourdomain.com/homelab`)
- **Password** — Require a password to access the destination URL
- **Description** — Add a note for your own reference
- **Expiration** — Set the link to expire after a specific time

## 2.4 Custom Domains

On the **Settings** page, you can add additional custom domains. Point your domain's DNS to the server running Kutt, and add the domain in settings. If you're using HTTPS through a reverse proxy, set `CUSTOM_DOMAIN_USE_HTTPS=true`.

## 2.5 API Access

Kutt provides a RESTful API for creating, managing, and deleting links programmatically. Generate an API key from the **Settings** page. The API can be used with integrations such as:

- **ShareX** — Set Kutt as your default URL shortener
- **Alfred Workflow** — Use the official [alfred-kutt](https://github.com/thedevs-network/alfred-kutt) workflow
- **iOS Shortcut** — Available for Apple devices via the sharing context menu
- **Browser Extension** — Available for Chrome and Firefox

## 2.6 Theming & Customization

Kutt supports custom CSS, images, and HTML templates. Place files in the `/kutt/custom` directory (mapped via Docker volume):

```
custom/
├─ css/
│   ├─ styles.css       ← replaces default styles
│   ├─ custom1.css      ← additional stylesheet
├─ images/
│   ├─ logo.png         ← replaces default logo
│   ├─ favicon.ico      ← replaces default favicon
├─ views/
│   ├─ partials/
│   │   ├─ footer.hbs
│   ├─ 404.hbs
```

> After making changes to the custom folder, restart the Kutt container for changes to take effect.
{.is-warning}

## 2.7 Making Your Instance Private

To lock down your Kutt instance for personal use, set these two environment variables:

```yaml
- DISALLOW_REGISTRATION=true
- DISALLOW_ANONYMOUS_LINKS=true
```

This prevents anyone else from signing up or creating short links without an account.

# <img src="/youtube.png" class="tab-icon"> 3 · Video

`[VIDEO EMBED: Add YouTube link here once video is published]`