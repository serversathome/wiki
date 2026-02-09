---
title: Transfer.zip
description: A guide to deploy Transfer.zip
published: true
date: 2026-02-09T16:55:58.472Z
tags: 
editor: markdown
dateCreated: 2026-02-09T16:52:40.763Z
---

# <img src="/transfer-zip.png" class="tab-icon"> What is Transfer.zip?

**Transfer.zip** is a self-hostable, open-source file-sharing solution and a privacy-focused alternative to services like WeTransfer and Smash. It supports two transfer modes: **Quick Transfers** using WebRTC peer-to-peer connections with end-to-end AES-256-GCM encryption (files never touch the server), and **Stored Transfers** using the resumable tus upload protocol for reliable, chunked uploads to server or S3-compatible storage. Transfer.zip also supports transfer requests, custom branding, and email notifications.


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Transfer.zip

Transfer.zip requires two repositories to function: `transfer.zip-web` (the frontend and API server) and `transfer.zip-node` (the file operations server). Both are built from source using Docker Compose.

> 
> Transfer.zip builds its Docker images from source rather than pulling pre-built images. Make sure you have `git` installed on your system.
{.is-info}

## 1.1 Clone the Repositories

```bash
cd /mnt/tank/stacks
git clone https://github.com/robinkarlberg/transfer.zip-web.git transfer-zip-web
git clone https://github.com/robinkarlberg/transfer.zip-node.git transfer-zip-node
```

## 1.2 Configure the Web Server

```bash
cd /mnt/tank/stacks/transfer-zip-web
cp .env.example .env
```

Edit the `.env` file and configure your domain and settings. You will also need to configure `next/.env` with any additional environment variables required for the Next.js frontend.

## 1.3 Deploy with Docker Compose

The default `docker-compose.yaml` in the `transfer-zip-web` repo deploys two services:

```yaml
services:
  next:
    build: next
    container_name: transfer-zip-web
    restart: unless-stopped
    env_file:
      - .env
      - next/.env
    ports:
      - "9001:9001"

  signaling-server:
    build: signaling-server
    container_name: transfer-zip-signaling
    restart: unless-stopped
    ports:
      - "9002:9002"
```

> 
> The `next` service runs the main web UI and API on port **9001**. The `signaling-server` handles WebRTC peer discovery on port **9002**. Both are required for Quick Transfers to function.
{.is-info}

## 1.4 Configure the Node Server (Stored Transfers)

If you want to support **Stored Transfers** (server-side file storage), you also need to deploy the `transfer-zip-node` server:

```bash
cd /mnt/tank/stacks/transfer-zip-node
./createenv.sh
```

Edit `server/conf.json` to configure your storage backend:
- **Disk storage** is enabled by default
- **S3-compatible storage**: Edit `server/conf.json` with your access keys and change the active provider

Generate a key pair for JWT authentication between the web and node servers, then place the public key in `./keys/public.pem` in the node repo. Update `next/conf.json` in the web repo to point to your node server's public URL.

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
| **Stored Transfer** | Resumable uploads to server/S3 | Depends on storage | No |
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