---
title: OpnForm
description: A guide to deploying OpnForm
published: true
date: 2026-10-01T16:13:23.436Z
tags: 
editor: markdown
dateCreated: 2026-09-28T16:23:10.223Z
---

# <img src="/opnform.png" class="tab-icon"> What is OpnForm?

**OpnForm** is an open-source, no-code form builder and a self-hosted alternative to Typeform and Google Forms. Build unlimited forms with conditional logic, file uploads, captcha protection and analytics, then embed them anywhere or share a link. Submissions can trigger email notifications, webhooks, or Slack and Discord messages, and the built-in MCP server lets AI assistants like Claude build forms and read results for you.

<div class="glance">
  <div><span>Port</span><b><code>8088</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>7 services</b></div>
  <div><span>Depends on</span><b>Postgres and Redis</b></div>
  <div><span>Difficulty</span><b class="difficulty intermediate">Intermediate</b></div>
  <div class="glance-links">
    <a href="https://github.com/OpnForm/OpnForm"><i class="mdi mdi-github"></i>Project</a>
    <a href="https://docs.opnform.com"><i class="mdi mdi-book-open-variant"></i>Docs</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy OpnForm

OpnForm is a Laravel API, a Nuxt front end, a queue worker, a scheduler, Postgres, Redis and an Nginx ingress that ties the front end and API together on one port. Everything you need to change lives in a single `.env` file: your URL, your secrets, email and MCP. The compose file reads from it, so you never have to edit the compose itself.

## 1.1 Create the folders and Nginx config

```bash
mkdir -p /mnt/tank/configs/opnform/{postgres,redis,storage}
chown -R 568:568 /mnt/tank/configs/opnform
```

Create `/mnt/tank/configs/opnform/nginx.conf` with the following contents. This is the upstream ingress config, unchanged:

```nginx
map $request_uri $api_uri {
    ~^/api(/.*$) $1;
    default $request_uri;
}

server {
    listen 80;
    server_name opnform;
    root /usr/share/nginx/html/public;
    client_max_body_size ${NGINX_MAX_BODY_SIZE};

    access_log /dev/stdout;
    error_log /dev/stderr error;

    index index.html index.htm index.php;

    location / {
        proxy_http_version 1.1;
        proxy_pass http://opnform-client:3000;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Port $server_port;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
    }

    location ~/(api|open|local\/temp|forms\/assets)/ {
        set $original_uri $uri;
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass opnform-api:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root/index.php;
        fastcgi_param REQUEST_URI $api_uri;
        fastcgi_param HTTP_X_FORWARDED_FOR $proxy_add_x_forwarded_for;
        fastcgi_param HTTP_X_FORWARDED_HOST $http_x_forwarded_host;
        fastcgi_param HTTP_X_FORWARDED_PORT $http_x_forwarded_port;
        fastcgi_param HTTP_X_FORWARDED_PROTO $http_x_forwarded_proto;
    }

    location ~ /\. {
        deny all;
    }
}
```

## 1.2 Generate secrets

Run this in the TrueNAS shell. It prints four lines that are already formatted for the `.env` file, so you can copy them straight across:

```bash
echo "APP_KEY=base64:$(openssl rand -base64 32)"
echo "JWT_SECRET=$(openssl rand -hex 20)"
echo "SHARED_SECRET=$(openssl rand -hex 20)"
echo "DB_PASSWORD=$(openssl rand -hex 16)"
```

## 1.3 The .env file

In Dockge, create a new stack called `opnform` and paste this into the **.env** editor, below the compose editor. This is the only place you make changes:

```bash
# ── URL ─────────────────────────────────────────────
# Your public address, with no trailing slash.
# Domain:   https://forms.example.com
# LAN only: http://192.168.1.10:8088
OPNFORM_URL=https://forms.example.com

# ── Secrets ─────────────────────────────────────────
# Paste the four lines from step 1.2 over these
APP_KEY=base64:CHANGEME
JWT_SECRET=CHANGEME
SHARED_SECRET=CHANGEME
DB_PASSWORD=CHANGEME

# ── Reverse proxy or tunnel ─────────────────────────
# IP or CIDR your proxy connects from. Leave blank if none. Never use *
TRUSTED_PROXIES=

# ── Email ───────────────────────────────────────────
# Leave MAIL_MAILER=log until your SMTP details are filled in.
# Change it to smtp to start sending real mail.
MAIL_MAILER=log
MAIL_HOST=smtp.example.com
MAIL_PORT=587
MAIL_USERNAME=forms@example.com
MAIL_PASSWORD='CHANGEME'
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=forms@example.com
MAIL_FROM_NAME=OpnForm

# ── MCP for AI assistants ───────────────────────────
# Needs an https OPNFORM_URL. See section 2.4
MCP_ENABLED=false
MCP_OAUTH_REDIRECT_DOMAINS=https://claude.ai,https://chatgpt.com,https://chat.openai.com,http://localhost,http://127.0.0.1,http://[::1]
```

> Wrap any value that contains a `$` in single quotes, like the mail password above. Without the quotes, Docker Compose treats the `$` as the start of a variable and silently mangles the value.
{.is-warning}

## 1.4 Compose file

Paste this into the compose editor exactly as it is. Every value comes from the `.env` file, so there is nothing to change here:

```yaml
x-api-env: &api-env
  APP_NAME: OpnForm
  APP_ENV: production
  APP_DEBUG: "false"
  APP_KEY: ${APP_KEY}
  APP_URL: ${OPNFORM_URL}
  FRONT_URL: ${OPNFORM_URL}
  FRONT_API_SECRET: ${SHARED_SECRET}
  JWT_SECRET: ${JWT_SECRET}
  JWT_TTL: "1440"
  SELF_HOSTED: "true"
  LOG_CHANNEL: errorlog
  LOG_LEVEL: info
  DB_CONNECTION: pgsql
  DB_HOST: db
  DB_PORT: "5432"
  DB_DATABASE: opnform
  DB_USERNAME: opnform
  DB_PASSWORD: ${DB_PASSWORD}
  REDIS_HOST: redis
  CACHE_DRIVER: redis
  CACHE_STORE: redis
  QUEUE_CONNECTION: redis
  SESSION_DRIVER: redis
  FILESYSTEM_DRIVER: local
  LOCAL_FILESYSTEM_VISIBILITY: public
  TRUSTED_PROXIES: ${TRUSTED_PROXIES}
  MAIL_MAILER: ${MAIL_MAILER}
  MAIL_HOST: ${MAIL_HOST}
  MAIL_PORT: ${MAIL_PORT}
  MAIL_USERNAME: ${MAIL_USERNAME}
  MAIL_PASSWORD: ${MAIL_PASSWORD}
  MAIL_ENCRYPTION: ${MAIL_ENCRYPTION}
  MAIL_FROM_ADDRESS: ${MAIL_FROM_ADDRESS}
  MAIL_FROM_NAME: ${MAIL_FROM_NAME}
  MCP_ENABLED: ${MCP_ENABLED}
  MCP_OAUTH_REDIRECT_DOMAINS: ${MCP_OAUTH_REDIRECT_DOMAINS}
  PHP_MEMORY_LIMIT: 1G
  PHP_MAX_EXECUTION_TIME: "600"
  PHP_UPLOAD_MAX_FILESIZE: 64M
  PHP_POST_MAX_SIZE: 64M

services:
  api:
    image: jhumanj/opnform-api:latest
    container_name: opnform-api
    environment: *api-env
    volumes:
      - /mnt/tank/configs/opnform/storage:/usr/share/nginx/html/storage
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped

  api-worker:
    image: jhumanj/opnform-api:latest
    container_name: opnform-api-worker
    command: ["php", "artisan", "queue:work"]
    environment: *api-env
    volumes:
      - /mnt/tank/configs/opnform/storage:/usr/share/nginx/html/storage
    depends_on:
      - api
    restart: unless-stopped

  api-scheduler:
    image: jhumanj/opnform-api:latest
    container_name: opnform-api-scheduler
    command: ["php", "artisan", "schedule:work"]
    environment: *api-env
    volumes:
      - /mnt/tank/configs/opnform/storage:/usr/share/nginx/html/storage
    depends_on:
      - api
    restart: unless-stopped

  ui:
    image: jhumanj/opnform-client:latest
    container_name: opnform-client
    user: "568:568"
    environment:
      NUXT_PUBLIC_APP_URL: ${OPNFORM_URL}
      NUXT_PUBLIC_API_BASE: ${OPNFORM_URL}/api
      NUXT_PRIVATE_API_BASE: http://ingress/api
      NUXT_API_SECRET: ${SHARED_SECRET}
      NUXT_PUBLIC_ENV: production
    depends_on:
      - api
    restart: unless-stopped

  redis:
    image: redis:7
    container_name: opnform-redis
    user: "568:568"
    volumes:
      - /mnt/tank/configs/opnform/redis:/data
    healthcheck:
      test: ["CMD-SHELL", "redis-cli ping | grep PONG"]
      interval: 30s
      timeout: 5s
    restart: unless-stopped

  db:
    image: postgres:16
    container_name: opnform-db
    user: "568:568"
    environment:
      POSTGRES_DB: opnform
      POSTGRES_USER: opnform
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - /mnt/tank/configs/opnform/postgres:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U opnform -d opnform"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  ingress:
    image: nginx:1
    container_name: opnform-ingress
    environment:
      - NGINX_MAX_BODY_SIZE=64m
    volumes:
      - /mnt/tank/configs/opnform/nginx.conf:/etc/nginx/templates/default.conf.template:ro
    ports:
      - "8088:80"
    depends_on:
      - api
      - ui
    restart: unless-stopped
```

1. Fill in the `.env` file: your URL, the four secrets, and email if you have SMTP ready
2. Deploy the stack and give the API a minute on first boot while it runs its database migrations

> The three API containers and the Nginx ingress do not get `user: "568:568"`. The API entrypoint writes PHP config into the image, creates its storage folders and chowns them to `www-data` before starting PHP-FPM, and the Nginx ingress renders its config template at startup. Both need to start as root. The storage folder will end up owned by `www-data` after the first boot, which is expected.
{.is-warning}

> Your URL is set once, in `OPNFORM_URL`, and the compose file fills in all four places OpnForm needs it. After any change to the `.env` file, click **Deploy** again in Dockge (or run `docker compose up -d`) so the containers are recreated. A plain restart does not pick up new values.
{.is-info}

# 2 · Configuration

## 2.1 Initial Setup

1. Browse to your `OPNFORM_URL`
2. OpnForm redirects to a setup page on first launch. Create your admin account there
3. Public registration shuts off once setup is complete. Invite anyone else from the admin account

> Free self-hosted instances are limited to **2 users** across the whole instance. A self-hosted Enterprise license lifts that cap and unlocks the Enterprise features, but the core form builder works without one.
{.is-info}

## 2.2 Email

Out of the box `MAIL_MAILER` is set to `log`, so notification emails land in the container logs instead of an inbox. To send real mail:

1. Fill in the `MAIL_` lines in the `.env` file with your SMTP provider's details
2. Change `MAIL_MAILER=log` to `MAIL_MAILER=smtp`
3. Redeploy the stack

Emails are sent by the `opnform-api-worker` container, not the API itself. If a test submission never arrives, check that container's logs first.

> Use an address on a domain you control for `MAIL_FROM_ADDRESS`, sent through that domain's own mail provider. That way it passes your existing SPF and DKIM records instead of landing in spam.
{.is-success}

## 2.3 Behind a Reverse Proxy or Tunnel

If Cloudflare Tunnel, Traefik or another proxy sits in front of the ingress, set `TRUSTED_PROXIES` in the `.env` file to the IP or CIDR the proxy connects from, then redeploy. This lets OpnForm see real client IPs for rate limiting.

```bash
TRUSTED_PROXIES=172.18.0.0/16
```

> Never set `TRUSTED_PROXIES` to `*`. That lets any client spoof its IP and bypass rate limits.
{.is-danger}

## 2.4 MCP for AI Assistants

OpnForm has a built-in MCP server. Once it is on, Claude, ChatGPT, Cursor and other MCP clients can build forms from a conversation, edit and publish them, and read and summarize your submissions. Every connection signs in with your OpnForm account through OAuth, so you never paste a password or token into a chat.

**Before you start:**

- [x] `OPNFORM_URL` is a public `https://` address, through a reverse proxy or Cloudflare Tunnel
- [x] The storage folder is persistent. The API generates its OAuth signing keys there on first boot (`oauth-private.key` and `oauth-public.key`)
- [x] `MCP_OAUTH_REDIRECT_DOMAINS` includes your AI client. The default list covers Claude, ChatGPT and local clients

**Turn it on:**

1. Set `MCP_ENABLED=true` in the `.env` file and redeploy, or skip this and use the switch in the next step
2. Sign in as the admin and open **Settings → MCP & AI agents**
3. Fix anything the setup checks flag, then turn on **Enable MCP**
4. Copy the MCP endpoint shown on that page. The page also has ready-to-copy setup for Claude Code, Cursor, ChatGPT and Codex

**Connect Claude:**

1. In Claude, open **Settings → Connectors → Add custom connector**
2. Paste your MCP endpoint and click **Connect**
3. Sign in to OpnForm and approve the connection on the consent screen
4. Start a new chat and ask Claude to build a form

> The switch in **Settings → MCP & AI agents** is saved in the database and overrides `MCP_ENABLED` from the `.env` file. If the two disagree, the settings page wins.
{.is-info}

> Never delete the storage folder or the two `oauth-*.key` files in it. New keys are generated on the next boot, and every AI client you have connected loses access and has to reconnect.
{.is-warning}

> Never run the MCP endpoint over plain `http`. Use it only through a proper `https` domain.
{.is-danger}

## 2.5 Captcha

Public forms are a spam target. OpnForm supports hCaptcha and reCAPTCHA. Add the site key to the client (`NUXT_PUBLIC_H_CAPTCHA_SITE_KEY`) and both keys to the API (`H_CAPTCHA_SITE_KEY`, `H_CAPTCHA_SECRET_KEY`), then enable captcha per form in the form settings.