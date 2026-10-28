---
title: Ente Photos
description: A guide to deploying Ente Photo
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:35.538Z
---

# ![](/ente-photos.png){class="tab-icon"} What is Ente Photos?

Ente Photos is the private, secure photo storage app with end-to-end encryption.

<div class="glance">
  <div><span>Port</span><b><code>3000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>5 services</b></div>
  <div><span>Depends on</span><b>Postgres</b></div>
  <div><span>Difficulty</span><b class="difficulty intermediate">Intermediate</b></div>
  <div class="glance-links">
    <a href="https://github.com/ente/ente"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Ente Photos
Ente Photos requires a normal docker compose file and in the **same folder** an additional file named `museum.yaml` to be present and configured. 

## 1.1 Docker Compose
```yaml
services:
  museum:
    image: ghcr.io/ente/server
    ports:
      - "${PORT}:8080" # API
    depends_on:
      postgres:
        condition: service_healthy
    volumes:
      - "./museum.yaml:/museum.yaml"
      - "${CONFIG_DIR}/data:/data"
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:8080/ping"]
      interval: 60s
      timeout: 5s
      retries: 3
      start_period: 120s

  # Resolve "localhost:3200" in the museum container to the minio container.
  socat:
    image: alpine/socat
    network_mode: service:museum
    depends_on:
      - museum
    command: "TCP-LISTEN:3200,fork,reuseaddr TCP:minio:3200"

  web:
    image: ghcr.io/ente/web
    ports:
      - "3000:3000" # Photos web app
      - "3002:3002" # Public albums
    environment:
      ENTE_API_ORIGIN: "${IP}:${PORT}"
      ENTE_ALBUMS_ORIGIN: "${IP}:3002"

  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: pguser
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ente_db
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -q -d ente_db -U pguser"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 40s
    volumes:
      - "${CONFIG_DIR}/postgres-data:/var/lib/postgresql/data"

  minio:
    image: pgsty/silo # MinIO-compatible, replaces the discontinued minio/minio image
    ports:
      - 3200:3200 # MinIO API
    environment:
      MINIO_ROOT_USER: ${MINIO_USER}
      MINIO_ROOT_PASSWORD: ${MINIO_PASSWORD}
    command: server /data --address ":3200" --console-address ":3201"
    volumes:
      - ${CONFIG_DIR}/minio-data:/data
    post_start:
      - command: |
          sh -c '
          #!/bin/sh

          while ! mc alias set h0 http://minio:3200 ${MINIO_USER} ${MINIO_PASSWORD} 2>/dev/null
          do
            echo "Waiting for minio..."
            sleep 0.5
          done

          cd /data

          mc mb -p b2-eu-cen
          mc mb -p wasabi-eu-central-2-v3
          mc mb -p scw-eu-fr-v3
          '
```

## 1.2 env File
```env
CONFIG_DIR=/mnt/tank/configs/ente
IP=http://10.99.0.242
PORT=8080
POSTGRES_PASSWORD=changeme
MINIO_USER=minio-user-changeme
MINIO_PASSWORD=changeme
```

> Don't reuse the example values. Generate your own with `openssl rand -base64 21 | tr -d '/+='` (run it once each for `POSTGRES_PASSWORD`, `MINIO_PASSWORD` and the suffix of `MINIO_USER`). You'll paste the same values into `museum.yaml` below.
{.is-warning}

## 1.3 Museum.yaml
> For `museum.yaml` below, replace `${IP}` with the address of your server!
{.is-danger}

> Replace every `<...>` placeholder with your own values. `<POSTGRES_PASSWORD>`, `<MINIO_USER>` and `<MINIO_PASSWORD>` must match your `.env` file. Generate the keys with:
> - `encryption`: `head -c 32 /dev/urandom | base64`
> - `hash`: `head -c 64 /dev/urandom | base64 | tr -d '\n'`
> - `jwt secret`: `head -c 32 /dev/urandom | base64 | tr '+/' '-_'`
{.is-warning}

```yaml
db:
      host: postgres
      port: 5432
      name: ente_db
      user: pguser
      password: <POSTGRES_PASSWORD>

s3:
      # Top-level configuration for buckets, you can override by specifying these configuration in the desired bucket.
      # Set this to false if using external object storage bucket or bucket with SSL
      are_local_buckets: true
      # Set this to false if using subdomain-style URL. This is set to true for ensuring compatibility with MinIO when SSL is enabled.
      use_path_style_urls: true
      b2-eu-cen:
         # Uncomment the below configuration to override the top-level configuration
         # are_local_buckets: true
         # use_path_style_urls: true
         key: <MINIO_USER>
         secret: <MINIO_PASSWORD>
         endpoint: ${IP}:3200
         region: eu-central-2
         bucket: b2-eu-cen
      wasabi-eu-central-2-v3:
         # are_local_buckets: true
         # use_path_style_urls: true
         key: <MINIO_USER>
         secret: <MINIO_PASSWORD>
         endpoint: ${IP}:3200
         region: eu-central-2
         bucket: wasabi-eu-central-2-v3
         compliance: false
      scw-eu-fr-v3:
         # are_local_buckets: true
         # use_path_style_urls: true
         key: <MINIO_USER>
         secret: <MINIO_PASSWORD>
         endpoint: ${IP}:3200
         region: eu-central-2
         bucket: scw-eu-fr-v3

# Specify the base endpoints for various web apps
apps:
    # If you're running a self hosted instance and wish to serve public links,
    # set this to the URL where your albums web app is running.
    public-albums: http://${IP}:3002
    cast: http://${IP}:3004
    # Set this to the URL where your accounts web app is running, primarily used for
    # passkey based 2FA.
    accounts: http://${IP}:3001
    # Optional apps (expose the matching port on the web service first)
    # embed-albums: http://${IP}:3006
    # public-locker: http://${IP}:3005
    # public-paste: http://${IP}:3008
    # public-memories: http://${IP}:3010

key:
      encryption: <generated-encryption-key>
      hash: <generated-hash-key>

jwt:
      secret: <generated-jwt-secret>
```
# 2 · Signing Up
1. Navigate to `http://{IP}:3000` 
1. Click **Don't Have An Account**
1. After you add an email and strong password click **Create Account**
1. Inspect the logs and look for this line to get your Verification Code:
    ```bash
    INFO[0074]email.go:202 sendViaTransmail Skipping sending email to test@test.com: Verification code: 068638
    ```
1. Click **Verify** after entering your code
1. Click "Continue with free plan 10 GB free forever"

# 3 · Removing Storage Limits
The official way is the [Ente CLI](https://github.com/ente/ente/tree/main/cli). The first account registered on your server is treated as admin until you [whitelist admins](https://ente.com/help/self-hosting/administration/users#whitelist-admins) in `museum.yaml`, so register your own account first.

```bash
ente account login --account home-server --app photos --host http://{IP}:8080
ente admin user update-subscription you@example.com --no-limit --account home-server
```

`--no-limit` sets 100 TB of storage and an expiry 100 years out. See the upstream [CLI administration docs](https://ente.com/help/cli/admin) for other options.

**Alternative (direct SQL):** in the TrueNAS shell, run this command **as root**:
```bash
docker exec -i ente-postgres-1 psql -U pguser -d ente_db -c "INSERT INTO storage_bonus (bonus_id, user_id, storage, type, valid_till) VALUES ('self-hosted-myself', (SELECT user_id FROM users LIMIT 1), 109951162777600, 'ADD_ON_SUPPORT', 0) ON CONFLICT (bonus_id) DO UPDATE SET storage = EXCLUDED.storage;"
```
