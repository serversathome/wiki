---
title: OpnForm
description: A guide to deploying OpnForm
published: true
date: 2026-09-28T16:23:10.223Z
tags: 
editor: markdown
dateCreated: 2026-09-28T16:23:10.223Z
---

# <img src="/opnform.png" class="tab-icon"> What is OpnForm?

**OpnForm** is an open-source, no-code form builder and a self-hosted alternative to Typeform and Google Forms. Build unlimited forms with conditional logic, file uploads, captcha protection and analytics, then embed them anywhere or share a link. Submissions can trigger email notifications, webhooks, or Slack and Discord messages.

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

OpnForm is a Laravel API, a Nuxt front end, a queue worker, a scheduler, Postgres, Redis and an Nginx ingress that ties the front end and API together on one port. The upstream compose expects two `.env` files from a cloned repo, so this version puts every variable directly in the compose file instead, which is what Dockge wants.

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

Run these in the TrueNAS shell and keep the output handy:

```bash
echo "APP_KEY=base64:$(openssl rand -base64 32)"
echo "JWT_SECRET=$(openssl rand -hex 20)"
echo "SHARED_SECRET=$(openssl rand -hex 20)"
echo "DB_PASSWORD=$(openssl rand -hex 16)"
```

`SHARED_SECRET` goes in two places: `FRONT_API_SECRET` on the API and `NUXT_API_SECRET` on the client. They must match.

## 1.3 Compose file

```yaml
x-api-env: &api-env
  APP_NAME: OpnForm
  APP_ENV: production
  APP_DEBUG: "false"
  APP_KEY: base64:CHANGEME
  APP_URL: http://your-server-ip:8088
  FRONT_URL: http://your-server-ip:8088
  FRONT_API_SECRET: CHANGEME_SHARED_SECRET
  JWT_SECRET: CHANGEME_JWT_SECRET
  JWT_TTL: "1440"
  SELF_HOSTED: "true"
  LOG_CHANNEL: errorlog
  LOG_LEVEL: info
  DB_CONNECTION: pgsql
  DB_HOST: db
  DB_PORT: "5432"
  DB_DATABASE: opnform
  DB_USERNAME: opnform
  DB_PASSWORD: CHANGEME_DB_PASSWORD
  REDIS_HOST: redis
  CACHE_DRIVER: redis
  CACHE_STORE: redis
  QUEUE_CONNECTION: redis
  SESSION_DRIVER: redis
  FILESYSTEM_DRIVER: local
  LOCAL_FILESYSTEM_VISIBILITY: public
  MAIL_MAILER: log
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
      NUXT_PUBLIC_APP_URL: http://your-server-ip:8088
      NUXT_PUBLIC_API_BASE: http://your-server-ip:8088/api
      NUXT_PRIVATE_API_BASE: http://ingress/api
      NUXT_API_SECRET: CHANGEME_SHARED_SECRET
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
      POSTGRES_PASSWORD: CHANGEME_DB_PASSWORD
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

1. Replace every `CHANGEME` value with the secrets you generated
2. Replace `your-server-ip` with your server's IP, or with your public hostname if OpnForm will sit behind a Cloudflare Tunnel (for example `https://forms.example.com` and `https://forms.example.com/api`)
3. Deploy the stack and give the API a minute on first boot while it runs its database migrations

> The three API containers and the Nginx ingress do not get `user: "568:568"`. The API entrypoint writes PHP config into the image, creates its storage folders and chowns them to `www-data` before starting PHP-FPM, and the Nginx ingress renders its config template at startup. Both need to start as root. The storage folder will end up owned by `www-data` after the first boot, which is expected.
{.is-warning}

> The public URL lives in four places: `APP_URL`, `FRONT_URL`, `NUXT_PUBLIC_APP_URL` and `NUXT_PUBLIC_API_BASE`. If you change it later, update all four and recreate the stack. A plain restart does not pick up new environment values.
{.is-info}

# 2 · Configuration

## 2.1 Initial Setup

1. Browse to `http://your-server-ip:8088`
2. OpnForm redirects to a setup page on first launch. Create your admin account there
3. Public registration shuts off once setup is complete. Invite anyone else from the admin account

> Free self-hosted instances are limited to **2 users** across the whole instance. A self-hosted Enterprise license lifts that cap and unlocks the Enterprise features, but the core form builder works without one.
{.is-info}

## 2.2 Email

Out of the box `MAIL_MAILER` is set to `log`, so notification emails land in the container logs instead of an inbox. To send real mail, add these to the `x-api-env` block and redeploy:

```yaml
  MAIL_MAILER: smtp
  MAIL_HOST: smtp.example.com
  MAIL_PORT: "587"
  MAIL_USERNAME: forms@example.com
  MAIL_PASSWORD: CHANGEME
  MAIL_ENCRYPTION: tls
  MAIL_FROM_ADDRESS: forms@example.com
  MAIL_FROM_NAME: OpnForm
```

## 2.3 Behind a Reverse Proxy or Tunnel

If Cloudflare Tunnel, Traefik or another proxy sits in front of the ingress, add `TRUSTED_PROXIES` to the `x-api-env` block with the IP or CIDR the proxy connects from. This lets OpnForm see real client IPs for rate limiting.

```yaml
  TRUSTED_PROXIES: 172.18.0.0/16
```

> Never set `TRUSTED_PROXIES` to `*`. That lets any client spoof its IP and bypass rate limits.
{.is-danger}

## 2.4 Captcha

Public forms are a spam target. OpnForm supports hCaptcha and reCAPTCHA. Add the site key to the client (`NUXT_PUBLIC_H_CAPTCHA_SITE_KEY`) and both keys to the API (`H_CAPTCHA_SITE_KEY`, `H_CAPTCHA_SECRET_KEY`), then enable captcha per form in the form settings.

