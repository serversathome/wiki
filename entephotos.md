---
title: Ente Photos
description: A guide to deploying Ente Photo
published: true
date: 2025-10-21T17:16:19.485Z
tags: 
editor: markdown
dateCreated: 2025-10-21T14:47:39.040Z
---

# ![](/ente-photos.png){class="tab-icon"} What is Ente Photos?

Ente Photos is the private, secure photo storage app with end-to-end encryption.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Ente Photos
Ente Photos requires a normal docker compose file and in the **same folder** an additional file named `museum.yaml` to be present and configured. 

> For both files below, replace my `10.99.0.242` IP address with the address of your server!
{.is-danger}


## 1.1 Docker Compose
```yaml
services:
  museum:
    image: ghcr.io/ente-io/server
    ports:
      - 8080:8080 # API
    depends_on:
      postgres:
        condition: service_healthy
    volumes:
      - ./museum.yaml:/museum.yaml:ro
      - ./data:/data:ro
    healthcheck:
      test: ["CMD", "curl", "--fail", "http://10.99.0.242:8080/ping"]
      interval: 60s
      timeout: 5s
      retries: 3
      start_period: 5s

  # Resolve "localhost:3200" in the museum container to the minio container.
  socat:
    image: alpine/socat
    network_mode: service:museum
    depends_on: [museum]
    command: "TCP-LISTEN:3200,fork,reuseaddr TCP:minio:3200"

  web:
    image: ghcr.io/ente-io/web
    # Uncomment what you need to tweak.
    ports:
      - 3000:3000 # Photos web app
      # - 3001:3001 # Accounts
      - 3002:3002 # Public albums
      # - 3003:3003 # Auth
      # - 3004:3004 # Cast
    # Modify these values to your custom subdomains, if using any
    environment:
      ENTE_API_ORIGIN: http://10.99.0.242:8080
      ENTE_ALBUMS_ORIGIN: https://10.99.0.242:3002

  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: pguser
      POSTGRES_PASSWORD: eXa6yOupIVyGXe02XnwTJX8ZjsfX
      POSTGRES_DB: ente_db
    healthcheck:
      test: pg_isready -q -d ente_db -U pguser
      start_period: 40s
      start_interval: 1s
    volumes:
      - ./postgres-data:/var/lib/postgresql/data

  minio:
    image: minio/minio
    ports:
      - 3200:3200 # MinIO API
      # Uncomment to enable MinIO Web UI      
      # - 3201:3201
    environment:
      MINIO_ROOT_USER: minio-user-Av/ztrFm
      MINIO_ROOT_PASSWORD: Jczt/BEywUms1wRKJ8BbaMmaxyGy
    command: server /data --address ":3200" --console-address ":3201"
    volumes:
      - ./minio-data:/data
    post_start:
      - command: |
          sh -c '
          #!/bin/sh

          while ! mc alias set h0 http://minio:3200 minio-user-Av/ztrFm Jczt/BEywUms1wRKJ8BbaMmaxyGy 2>/dev/null
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

## Museum.yaml

```yaml
db:
      host: postgres
      port: 5432
      name: ente_db
      user: pguser
      password: eXa6yOupIVyGXe02XnwTJX8ZjsfX

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
         key: minio-user-Av/ztrFm
         secret: Jczt/BEywUms1wRKJ8BbaMmaxyGy
         endpoint: 10.99.0.242:3200
         region: eu-central-2
         bucket: b2-eu-cen
      wasabi-eu-central-2-v3:
         # are_local_buckets: true
         # use_path_style_urls: true
         key: minio-user-Av/ztrFm
         secret: Jczt/BEywUms1wRKJ8BbaMmaxyGy
         endpoint: 10.99.0.242:3200
         region: eu-central-2
         bucket: wasabi-eu-central-2-v3
         compliance: false
      scw-eu-fr-v3:
         # are_local_buckets: true
         # use_path_style_urls: true
         key: minio-user-Av/ztrFm
         secret: Jczt/BEywUms1wRKJ8BbaMmaxyGy
         endpoint: 10.99.0.242:3200
         region: eu-central-2
         bucket: scw-eu-fr-v3

# Specify the base endpoints for various web apps
apps:
    # If you're running a self hosted instance and wish to serve public links,
    # set this to the URL where your albums web app is running.
    public-albums: http://10.99.0.242:3002
    cast: http://10.99.0.242:3004
    # Set this to the URL where your accounts web app is running, primarily used for
    # passkey based 2FA.
    accounts: http://10.99.0.242:3001

key:
      encryption: OSszWn1D13RxVsLp4b5TZuICvlt9JSWKwyP4YpFxCDc=
      hash: xvSPIxUuZ8Rmn2EHlDHMYqIM6w64niPbFjbGscUmCfcIHo4txT/6eXIro0oxPlkLuWrEBMS28Ctom+f0Klifrw==

jwt:
      secret: rV48t6COlobRUUUWUnD8vSdsXA_dXa2s_Xmt9A_TCqM=
```
# 2 · Signing Up
1. Navigate to http://{IP}:3000 
1. Click **Register**
1. After you add an email and strong password click **Sign Up**
1. Inspect the logs and look for this line to get your Verification Code:
```bash
INFO[0074]email.go:202 sendViaTransmail Skipping sending email to test@test.com: Verification code: 068638
```