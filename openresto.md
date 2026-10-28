---
title: Openresto
description: A guide to deploying Openresto
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-06-29T20:52:10.263Z
---

# What is OpenResto?

**OpenResto** is a self-hosted, open source restaurant booking management system. Customers browse restaurants, hold a table in real time, and book instantly from a clean mobile friendly UI with no ads and no account required. Owners manage reservations, tables, floor sections, branding, and booking pauses from a dedicated admin dashboard.

It is built for indie restaurants, bars, and cafes that do not want to pay SaaS booking fees or hand customer data to a third party. Everything runs from a single Docker Compose stack with no external services required beyond optional SMTP for confirmation emails.

<div class="glance">
  <div><span>Port</span><b><code>8080</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>3 services</b></div>
  <div><span>Difficulty</span><b class="difficulty intermediate">Intermediate</b></div>
  <div class="glance-links">
    <a href="https://github.com/karanshukla/openresto"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>



# <img src="/docker.png" class="tab-icon"> 1 · Deploy OpenResto

```yaml
services:
  backend:
    image: ghcr.io/karanshukla/openresto-backend:latest
    container_name: openresto-backend
    env_file: .env # passes SMTP and other optional settings
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - TZ=America/New_York
      - PORT=8080
      - CORS_ORIGINS=https://${DOMAIN_NAME}
      - JWT_KEY=${JWT_KEY}
      - ADMIN_EMAIL=${ADMIN_EMAIL}
      - ADMIN_PASSWORD=${ADMIN_PASSWORD}
      - CONNECTION_STRING=${CONNECTION_STRING:-Data Source=/data/openresto.db}
      - DATA_PROTECTION_KEYS_PATH=/data/dp-keys
      - Vapid__PublicKey=${VAPID_PUBLIC_KEY}
      - Vapid__PrivateKey=${VAPID_PRIVATE_KEY}
      - Vapid__Subject=${VAPID_SUBJECT}
    volumes:
      - /mnt/tank/configs/openresto/data:/data
      - /mnt/tank/configs/openresto/media:/app/wwwroot/media
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/api/health"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 20s
    restart: always

  frontend:
    image: ghcr.io/karanshukla/openresto-frontend:latest
    container_name: openresto-frontend
    environment:
      - EXPO_PUBLIC_API_URL=/api
      - PORT=8081
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8081"]
      interval: 10s
      timeout: 5s
      retries: 3
    restart: always

  reverse-proxy:
    image: ghcr.io/karanshukla/openresto-nginx:latest
    container_name: openresto-proxy
    ports:
      - "127.0.0.1:8080:80"
      - "127.0.0.1:8443:443"
    environment:
      - DOMAIN_NAME=${DOMAIN_NAME}
      - BACKEND_HOST=backend
      - BACKEND_PORT=8080
      - FRONTEND_HOST=frontend
      - FRONTEND_PORT=8081
      - SSL_CERT_PATH=/etc/nginx/ssl/server.crt
      - SSL_KEY_PATH=/etc/nginx/ssl/server.key
    volumes:
      - ${HOST_SSL_CERT_PATH}:/etc/nginx/ssl/server.crt:ro
      - ${HOST_SSL_KEY_PATH}:/etc/nginx/ssl/server.key:ro
      - /mnt/tank/configs/openresto/media:/usr/share/nginx/html/media:ro
    depends_on:
      backend:
        condition: service_healthy
      frontend:
        condition: service_healthy
    restart: always
```


> 
> The `reverse-proxy` is not optional. The frontend image is built to call the API at `/api`, so Nginx does the path routing: `/api` goes to the backend and everything else goes to the frontend. It also terminates TLS using the certificates you mount from the host. Removing it breaks all API calls.
{.is-warning}

> 
> The proxy binds only to `127.0.0.1` on ports `8080` (HTTP) and `8443` (HTTPS), so it is not exposed to your LAN directly. That makes it a clean target for a Cloudflare Tunnel or a separate reverse proxy.
{.is-info}



# 2 · Configuration

## 2.1 Environment file

Create a `.env` file next to the compose file:

```env
# OpenResto Configuration

# --- STACK ---
# Public hostname, used for CORS and the proxy config
DOMAIN_NAME=booking.yourdomain.com
# Host paths to your TLS certificate and key (see section 2.3)
HOST_SSL_CERT_PATH=/mnt/tank/configs/openresto/ssl/server.crt
HOST_SSL_KEY_PATH=/mnt/tank/configs/openresto/ssl/server.key

# --- BACKEND ---
# JWT signing key — REQUIRED, min 32 chars, must be unique per deployment.
# Generate one with: openssl rand -base64 48
JWT_KEY=your_secure_random_key_here_min_32_chars

# SQLite connection string — keep it under /data so the database lives on the bind mount
CONNECTION_STRING=Data Source=/data/openresto.db

# CORS origins are set in the compose file from DOMAIN_NAME (https://${DOMAIN_NAME})

# Default UI language: en, fr, es or de
OPENRESTO_DEFAULT_LOCALE=en

# Initial Admin Credentials — REQUIRED on first boot (only used to seed the DB if empty).
ADMIN_EMAIL=admin@your-domain.com
# Generate a strong password with: openssl rand -base64 24
ADMIN_PASSWORD=change-me-to-a-strong-password

# VAPID Push Notifications (leave empty to disable — app degrades gracefully)
# Generate keys with: npx web-push generate-vapid-keys
VAPID_PUBLIC_KEY=
VAPID_PRIVATE_KEY=
VAPID_SUBJECT=mailto:you@your-domain.com

# SMTP Settings for Email Notifications
EmailSettings__Host=smtp.example.com
EmailSettings__Port=587
EmailSettings__Username=user@example.com
EmailSettings__Password=your_smtp_password
EmailSettings__FromEmail=noreply@example.com
EmailSettings__FromName=OpenResto
EmailSettings__EnableSsl=true

```

> The `.env` file in the upstream repo uses `CONNECTION_STRING=Data Source=./openresto.db`. Don't copy that line: the `.env` value wins over the compose default, and `./openresto.db` puts the database inside the container, where it is lost every time the container is recreated.
{.is-danger}

| Variable | Description |
|----------|-------------|
| `DOMAIN_NAME` | Your public hostname, e.g. `booking.yourdomain.com`. Used for CORS and the proxy config |
| `JWT_KEY` | JWT signing key, HS256, minimum 32 characters. Generate a strong random value |
| `ADMIN_EMAIL` | Default admin login email |
| `ADMIN_PASSWORD` | Default admin login password |
| `CONNECTION_STRING` | Optional. SQLite connection string. Must point under `/data`, e.g. `Data Source=/data/openresto.db` |
| `VAPID_PUBLIC_KEY` | VAPID public key for web push notifications |
| `VAPID_PRIVATE_KEY` | VAPID private key for web push notifications |
| `VAPID_SUBJECT` | VAPID subject, a `mailto:` or `https:` contact URI, e.g. `mailto:admin@yourdomain.com` |
| `HOST_SSL_CERT_PATH` | Host path to your TLS certificate, mounted into the proxy |
| `HOST_SSL_KEY_PATH` | Host path to your TLS private key, mounted into the proxy |
{.dense}

> 
> Generate a VAPID keypair with `npx web-push generate-vapid-keys`. Web push lets the admin dashboard receive booking notifications in the browser. If you do not need push notifications you can leave the Vapid values blank, but generating a keypair takes seconds.
{.is-info}

## 2.2 How the config keys map

OpenResto reads a mix of flat and nested keys. The flat ones (`PORT`, `JWT_KEY`, `ADMIN_EMAIL`, `ADMIN_PASSWORD`, `CONNECTION_STRING`, `CORS_ORIGINS`) are read directly. The push keys use ASP.NET Core's nested binding, where the appsettings section `Vapid:PublicKey` becomes the environment variable `Vapid__PublicKey` with a double underscore. The compose file maps the flat `VAPID_PUBLIC_KEY`, `VAPID_PRIVATE_KEY` and `VAPID_SUBJECT` values from `.env` onto those nested keys, so use the flat names in `.env`. Other nested settings in `.env` (such as `EmailSettings__*`) reach the backend through `env_file`.

## 2.3 TLS certificates

The proxy expects a certificate and key mounted read-only from the host at the paths in `HOST_SSL_CERT_PATH` and `HOST_SSL_KEY_PATH`. Options:

- Provide a real certificate (Let's Encrypt, or one issued by your own CA).
- If you front OpenResto with Cloudflare Tunnel or another proxy that already handles TLS, you can supply a self-signed certificate here just to satisfy the container, and let your edge handle the public certificate.

## 2.4 First login

1. Browse to your OpenResto URL.
2. Open the admin dashboard and log in with the `ADMIN_EMAIL` and `ADMIN_PASSWORD` you set.
3. Create your first restaurant, add tables and floor sections, then set your branding (app name, primary color, favicon icon).

> 
> Change the default admin password immediately after your first login.
{.is-danger}

