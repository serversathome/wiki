---
title: WYGIWYH
description: A guide to deploying WYGIWYH
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:09:45.123Z
---

# <img src="/wygiwyh.png" class="tab-icon"> What is WYGIWYH?
WYGIWYH (What You Get Is What You Have) is a powerful, principles-first finance tracker designed for people who prefer a no-budget, straightforward approach to managing their money. With features like multi-currency support, customizable transactions, and a built-in dollar-cost averaging tracker, WYGIWYH helps you take control of your finances with simplicity and flexibility.

<div class="glance">
  <div><span>Port</span><b><code>9005</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>2 services</b></div>
  <div><span>Depends on</span><b>Postgres</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy WYGIWYH
```yaml
services:
  web:
    image: eitchtee/wygiwyh:latest
    container_name: ${SERVER_NAME}
    ports:
      - "${OUTBOUND_PORT:-8000}:${INTERNAL_PORT:-8000}"
    env_file:
      - .env
    volumes:
      - /mnt/tank/configs/wygiwyh/attachments:/usr/src/app/attachments/
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: postgres:15
    container_name: ${DB_NAME}
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/wygiwyh/db:/var/lib/postgresql/data/
    environment:
      - POSTGRES_USER=${SQL_USER}
      - POSTGRES_PASSWORD=${SQL_PASSWORD}
      - POSTGRES_DB=${SQL_DATABASE}
```

## env File
```env
SERVER_NAME=wygiwyh_server
DB_NAME=wygiwyh_pg

TZ=UTC # Change to your timezone. This only affects some async tasks.

DEBUG=false
URL=http://YOUR-SERVER-IP:9005 # Every address you open WYGIWYH at, separated by spaces (e.g. add https://wygiwyh.yourdomain.com)
HTTPS_ENABLED=false # Set to true only when you reach WYGIWYH over HTTPS (behind a reverse proxy)
SECRET_KEY=<GENERATE A SAFE SECRET KEY AND PLACE IT HERE> # e.g. openssl rand -hex 32
DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1] YOUR-SERVER-IP # Add your domain here too if you use one
OUTBOUND_PORT=9005

# Uncomment these variables to automatically create an admin account using these credentials on startup.
# After your first successfull login you can remove these variables from your file for safety reasons.
#ADMIN_EMAIL=<ENTER YOUR EMAIL>
#ADMIN_PASSWORD=<YOUR SAFE PASSWORD>

SQL_DATABASE=wygiwyh
SQL_USER=wygiwyh
SQL_PASSWORD=<INSERT A SAFE PASSWORD HERE>
SQL_HOST=${DB_NAME}
SQL_PORT=5432

# Gunicorn
WEB_CONCURRENCY=4

# App Configs
# Enable this if you want to keep deleted transactions in the database
ENABLE_SOFT_DELETE=false
# If ENABLE_SOFT_DELETE is true, transactions deleted for more than KEEP_DELETED_TRANSACTIONS_FOR days will be truly deleted. Set to 0 to keep all.
KEEP_DELETED_TRANSACTIONS_FOR=365

TASK_WORKERS=1 # This only work if you're using the single container option. Increase to have more open queues via procrastinate, you probably don't need to increase this.

# OIDC Configuration. Uncomment the lines below if you want to add OIDC login to your instance
#OIDC_CLIENT_NAME=""
#OIDC_CLIENT_ID=""
#OIDC_CLIENT_SECRET=""
#OIDC_SERVER_URL=""
#OIDC_ALLOW_SIGNUP=true
```

1. Generate a `SECRET_KEY` with `openssl rand -hex 32` and paste the output into the `.env` file.
1. To create an admin account on first start, uncomment `ADMIN_EMAIL` and `ADMIN_PASSWORD` and fill in your own values. Remove them again after your first login.
1. Replace `YOUR-SERVER-IP` in `URL` and `DJANGO_ALLOWED_HOSTS` with your server's IP address. Django rejects any address that isn't in `DJANGO_ALLOWED_HOSTS` (a 400 error), and form posts from any origin that isn't in `URL` fail the CSRF check.
1. The UI is published on host port `9005` (set by `OUTBOUND_PORT`), so open `http://YOUR-SERVER-IP:9005`.

> `HTTPS_ENABLED=false` is for plain-HTTP access on your LAN. With `true`, the login cookie is marked secure and browsers won't send it over `http://`, so you can't log in. Once WYGIWYH is behind a reverse proxy with HTTPS, set `HTTPS_ENABLED=true` and add your `https://` domain to `URL` and the domain to `DJANGO_ALLOWED_HOSTS`.
{.is-warning}

> The `attachments` volume keeps uploaded transaction attachments. Without it, they are lost whenever the container is recreated.
{.is-info}
