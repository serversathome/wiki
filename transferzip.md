---
title: Transfer.zip
description: A guide to deploy Transfer.zip
published: true
date: 2026-02-09T19:33:38.049Z
tags: 
editor: markdown
dateCreated: 2026-02-09T16:52:40.763Z
---

# <img src="/transfer-zip.png" class="tab-icon"> What is Transfer.zip?

**Transfer.zip** is a self-hostable, open-source file-sharing solution and a privacy-focused alternative to services like WeTransfer and Smash. It supports two transfer modes: **Quick Transfers** using WebRTC peer-to-peer connections with end-to-end AES-256-GCM encryption (files never touch the server), and **Stored Transfers** using the resumable tus upload protocol for reliable, chunked uploads to server or S3-compatible storage. Transfer.zip also supports transfer requests, custom branding, and email notifications.



# <img src="/docker.png" class="tab-icon"> 1 · Deploy Transfer.zip

The `transfer.zip-web` repository contains everything needed to self-host: the Next.js frontend, signaling server, worker, and MongoDB — all deployed via a single Docker Compose stack.

> 
> Transfer.zip builds its Docker images from source rather than pulling pre-built images. Make sure you have `git` installed on your system.
{.is-info}

## 1.1 Clone the Repository

```bash
cd /mnt/tank/configs
git clone https://github.com/robinkarlberg/transfer.zip-web.git transfer-zip-web
```

## 1.2 Configure Environment Files

```bash
cd /mnt/tank/configs/transfer-zip-web
cp .env.example .env
cp next/.env.example next/.env
```

Edit `.env` and `next/.env` to configure your domain, MongoDB credentials, and other settings.

Next, copy the example configuration file for the Next.js frontend. The Docker build will fail without this:

```bash
cp next/conf.json.example next/conf.json
```

> 
> The build will fail with a `"/conf.json": not found` error if you skip this step.
{.is-warning}

## 1.3 Generate JWT Keys

Transfer.zip uses JWT authentication between the API server and the worker. You need to generate a key pair and place the public key where the worker can find it:

```bash
cd /mnt/tank/configs/transfer-zip-web
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem
```

The private key is used by the `next` service (configure the path in your `.env` or `next/.env`). The public key needs to be placed into the worker's data volume. After the first deploy, copy it in:

```bash
docker compose up -d
docker cp public.pem transfer-zip-web-worker-1:/worker_data/public.pem
docker compose restart worker
```

> 
> The worker will crash with `ENOENT: no such file or directory, open '/worker_data/public.pem'` until the public key is in place. This is expected on first deploy.
{.is-warning}

## 1.4 Docker Compose Overview

The `docker-compose.yaml` deploys four services:

```yaml
services:
  mongo:
    image: mongo:8.0-noble
    restart: unless-stopped
    volumes:
      - ./_db:/data/db
    ports:
      - 127.0.0.1:${MONGODB_FORWARD_PORT:-27017}:27017
    env_file:
      - .env

  next:
    build: next
    restart: unless-stopped
    env_file:
      - .env
      - next/.env
    ports:
      - 127.0.0.1:${WEB_SERVER_FORWARD_PORT:-9001}:9001

  signaling-server:
    build: signaling-server
    restart: unless-stopped
    ports:
      - 127.0.0.1:${SIGNALING_SERVER_FORWARD_PORT:-9002}:9002

  worker:
    build: worker
    restart: unless-stopped
    env_file:
      - .env
    volumes:
      - worker_data:/worker_data

volumes:
  worker_data: {}
```

| Service | Purpose | Default Port |
|---------|---------|-------------|
| **mongo** | MongoDB database for user data and transfer metadata | 27017 |
| **next** | Web UI and API server (Next.js) | 9001 |
| **signaling-server** | WebRTC signaling and relay for Quick Transfers | 9002 |
| **worker** | Handles file operations for Stored Transfers | — |
{.dense}

> 
> All ports bind to `127.0.0.1` by default, meaning they are only accessible locally. You will need a reverse proxy or tunnel (see section 2.1) to expose the service externally.
{.is-info}

> 
> If your CPU does not support AVX instructions, use `mongo:4.4` instead of `mongo:8.0-noble`.
{.is-warning}

# 2 · Configuration

## 2.1 Reverse Proxy Setup

Transfer.zip requires a reverse proxy to serve both the web UI and the WebSocket signaling server. The `/ws` path must be routed to the signaling server.

### Nginx

```nginx
# Place this at the top of your nginx config
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

# Inside your server block
location / {
    proxy_buffering off;
    proxy_pass http://localhost:9001/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}

location /ws {
    proxy_pass http://localhost:9002/;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

### Apache

```apache
ProxyPreserveHost On

ProxyPass /ws ws://localhost:9002/
ProxyPassReverse /ws ws://localhost:9002/

ProxyPass / http://localhost:9001/
ProxyPassReverse / http://localhost:9001/
```

### Built-in Caddy

Transfer.zip includes a Caddy configuration for automatic SSL. Edit the `.env` file and set `CADDY_DOMAIN` to your domain, then deploy using:

```bash
./deploy-caddy.sh
```

> 
> The Caddy container listens on `0.0.0.0` by default. Make sure to firewall it if you do not want it exposed directly to the internet.
{.is-warning}

### Traefik

If you use Traefik, add the appropriate labels to your `docker-compose.yaml`:

```yaml
services:
  next:
    build: next
    restart: unless-stopped
    env_file:
      - .env
      - next/.env
    networks:
      - traefik
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.transferzip.rule=Host(`transferzip.example.com`)"
      - "traefik.http.routers.transferzip.entrypoints=websecure"
      - "traefik.http.routers.transferzip.tls.certresolver=myresolver"
      - "traefik.http.services.transferzip.loadbalancer.server.port=9001"

  signaling-server:
    build: signaling-server
    restart: unless-stopped
    networks:
      - traefik
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.transferzip-ws.rule=Host(`transferzip.example.com`) && PathPrefix(`/ws`)"
      - "traefik.http.routers.transferzip-ws.entrypoints=websecure"
      - "traefik.http.routers.transferzip-ws.tls.certresolver=myresolver"
      - "traefik.http.routers.transferzip-ws.service=transferzip-ws"
      - "traefik.http.services.transferzip-ws.loadbalancer.server.port=9002"
      - "traefik.http.services.transferzip-ws.loadbalancer.server.scheme=http"
```

> 
> The `scheme=http` label is **required** for the signaling server service in Traefik.
{.is-danger}

### Cloudflare Tunnels

If you already have a Cloudflare Tunnel connector running on your system, add two **public hostnames** in the Zero Trust dashboard under **Networks → Tunnels → Your Tunnel → Public Hostname**.

Add the `/ws` route **first**, then the catch-all:

| Subdomain | Domain | Path | Type | URL |
|-----------|--------|------|------|-----|
| transfer | yourdomain.com | /ws | HTTP | localhost:9002 |
| transfer | yourdomain.com | | HTTP | localhost:9001 |
{.dense}

> 
> The `/ws` entry **must** be listed above the catch-all entry in your tunnel's public hostname list. Cloudflare evaluates routes top-down and the signaling server needs to match first. Cloudflare handles WebSocket upgrades automatically.
{.is-warning}

> 
> Cloudflare's free plan has a 100MB upload limit per request. Quick Transfers using direct WebRTC bypass the tunnel entirely, but if the connection falls back to relay mode, traffic routes through the signaling server and the tunnel — which may hit this limit for larger files.
{.is-info}

## 2.2 Transfer Modes

| Mode | How It Works | File Size Limit | Requires Both Online |
|------|-------------|-----------------|---------------------|
| **Quick Transfer** | WebRTC P2P with AES-256-GCM E2E encryption | No limit | Yes |
| **Stored Transfer** | Resumable tus uploads to server/S3 | Depends on storage | No |
{.dense}

**Quick Transfers** stream files directly between browsers. If a direct WebRTC connection cannot be established (due to firewalls), the signaling server acts as a relay. For files larger than 10MB, the relay is forced for performance reasons.

**Stored Transfers** upload files to the server using the tus protocol, which supports resumable and chunked uploads. Files are deleted after the transfer expiry date.

## 2.3 Custom Branding

Transfer.zip supports custom branding for transfer pages, including custom icons and backgrounds. This feature requires an S3-compatible storage bucket. Configure the `S3_` environment variables in `next/.env` to enable this.

## 2.4 Known Limitations

- Sending files via Quick Transfer does not currently work on **Firefox Mobile**
- Some **Safari** browsers may have issues with WebSocket connections when the window is unfocused



# <img src="/youtube.png" class="tab-icon"> 3 · Video

*Coming soon*