---
title: Transfer.zip
description: A guide to deploy Transfer.zip
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-02-09T16:52:40.763Z
---

# <img src="/transfer-zip.png" class="tab-icon"> What is Transfer.zip?

**Transfer.zip** is a self-hostable, open-source file-sharing solution and a privacy-focused alternative to services like WeTransfer and Smash. It supports two transfer modes: **Quick Transfers** using WebRTC peer-to-peer connections with end-to-end AES-256-GCM encryption (files never touch the server), and **Stored Transfers** using the resumable tus upload protocol for reliable, chunked uploads to server or S3-compatible storage. Transfer.zip also supports transfer requests, custom branding, and email notifications.

<div class="glance">
  <div><span>Port</span><b><code>9001</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>4 services</b></div>
  <div><span>Difficulty</span><b class="difficulty advanced">Advanced</b></div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Transfer.zip

The `transfer.zip-web` repository contains everything needed to self-host: the Next.js frontend, signaling server, worker, and MongoDB — all deployed via a single Docker Compose stack.

> 
> Transfer.zip builds its Docker images from source rather than pulling pre-built images. Make sure you have `git` installed on your system.
{.is-info}

## 1.0 Automated Deployment (Recommended)

A deployment script is available that handles the entire setup automatically — cloning the repo, generating environment files, fixing known build issues, detecting your host IP, deploying all four containers, and checking the worker's key pair.

Shell into your TrueNAS system and navigate to your stacks directory:

```bash
cd /mnt/tank/stacks
```

Download and run the script:

```bash
curl -fsSL https://raw.githubusercontent.com/serversathome/ServersatHome/main/transfer-zip.sh -o transfer-zip.sh
chmod +x transfer-zip.sh
./transfer-zip.sh
```

The script will:
1. Auto-detect your TrueNAS host private IPv4
2. Prompt you for a domain (or default to LAN-only access)
3. Clone the repository and run `createenv.sh`
4. Copy `conf.json.example` and patch the Dockerfile permissions bug
5. Bind the web (9001) and signaling (9002) ports to your IP, and update `next/.env` with your IP and domain. MongoDB stays bound to `127.0.0.1`.
6. Build and deploy all containers and verify services
7. Check the worker's own key pair (regenerating it if it's missing) and print the worker's public key, which you need for Stored Transfers

Once complete, create your account as shown in [1.6 Create an Account](#h-16-create-an-account), then go to [Section 2 · Configuration](#h-2-configuration) for reverse proxy setup.

> 
> If you prefer to deploy manually or need to customize the process, follow sections 1.1 through 1.6 below instead.
{.is-info}

## 1.1 Clone the Repository

```bash
cd /mnt/tank/stacks
git clone https://github.com/robinkarlberg/transfer.zip-web.git transfer-zip-web
```

## 1.2 Configure Environment Files

Run upstream's `createenv.sh`. It copies `.env`, `next/.env` and `next/conf.json` from their examples and sets a random `MONGO_INITDB_ROOT_PASSWORD`:

```bash
cd /mnt/tank/stacks/transfer-zip-web
./createenv.sh
```

Then edit `.env` and `next/.env` to configure your domain and other settings.

> 
> If you copy the example files by hand instead (`cp .env.example .env`, `cp next/.env.example next/.env`, `cp next/conf.json.example next/conf.json`), change `MONGO_INITDB_ROOT_PASSWORD` in `.env` yourself, or MongoDB is left with the example password `changeme`. `createenv.sh` stops without changing anything if `.env` already exists.
{.is-info}

> 
> The build will fail with a `"/conf.json": not found` error if you skip this step.
{.is-warning}

## 1.3 Fix Dockerfile Permissions

The default Dockerfile for the `next` service has a permission issue where the `public` directory is copied as root but the container runs as the `nextjs` user. Fix this before building:

```bash
cd /mnt/tank/stacks/transfer-zip-web
sed -i 's|COPY --from=builder /app/public ./public|COPY --from=builder --chown=nextjs:nodejs /app/public ./public|' next/Dockerfile
```

> 
> Without this fix, the `next` service will enter a restart loop with `EACCES: permission denied, scandir '/app/public/api'` errors.
{.is-warning}

## 1.4 Deploy and Get the Worker's Public Key

Build and start the stack:

```bash
cd /mnt/tank/stacks/transfer-zip-web
docker compose up -d --build
```

Transfer.zip uses JWT authentication between the worker and the API servers. You don't need to create any keys: the worker image generates its own key pair in `/worker_data` (the `worker_data` volume), signs tokens with `private.pem`, and prints `public.pem` on startup. To read the public key (you need it for Stored Transfers, see [2.2](#h-22-transfer-modes)):

```bash
docker compose exec worker cat /worker_data/public.pem
```

> 
> Don't generate keys with `openssl` or copy a `public.pem` into the worker volume by hand. The public key must match the worker's own `private.pem`, or tokens can't be verified.
{.is-warning}

> 
> If the worker crashes with `ENOENT: no such file or directory, open '/worker_data/public.pem'` (an old or empty `worker_data` volume), regenerate the key pair and restart the worker:
> ```bash
> docker compose run --rm --no-deps -u 0 worker /app/docker-create-keys.sh
> docker compose restart worker
> ```
{.is-info}

## 1.5 Docker Compose Overview

The `docker-compose.yml` deploys four services:

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

## 1.6 Create an Account

Account creation is not supported in the UI when self-hosting. With the stack running, create your account from the repository folder:

```bash
cd /mnt/tank/stacks/transfer-zip-web
./create-account.sh
```

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

If you use Traefik, add the appropriate labels to your `docker-compose.yml`:

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

> Upstream's self-hosting guide says Stored Transfers need a separate [transfer.zip-node](https://github.com/robinkarlberg/transfer.zip-node) server, which handles the uploads and zip bundles. In that flow you run the main `worker` once to print its public key, paste it into `keys/public.pem` in the node repo, deploy the node, and then add the node's public URL to `next/conf.json` on the main server. If Stored Transfers don't work with the setup above, follow the [upstream guide](https://github.com/robinkarlberg/transfer.zip-web/blob/main/SELFHOSTING.md#setting-up-stored-transfers). Quick Transfers work without it.
{.is-info}

## 2.3 Custom Branding

Transfer.zip supports custom branding for transfer pages, including custom icons and backgrounds. This feature requires an S3-compatible storage bucket. Configure the `S3_` environment variables in `next/.env` to enable this.

## 2.4 HTTP / LAN-Only Access

If you are accessing Transfer.zip over plain HTTP (no reverse proxy or tunnel with HTTPS), sign-in will silently fail because the auth cookie is set with the `Secure` flag. To fix this for LAN use:

```bash
cd /mnt/tank/stacks/transfer-zip-web
sed -i 's|secure: !IS_DEV,|secure: false,|' next/src/lib/server/serverUtils.js
docker compose up -d --build next
```

> 
> If you later add HTTPS via a reverse proxy or Cloudflare Tunnel, revert this change and rebuild: `sed -i 's|secure: false,|secure: !IS_DEV,|' next/src/lib/server/serverUtils.js`
{.is-info}

## 2.5 Known Limitations

- Sending files via Quick Transfer does not currently work on **Firefox Mobile**
- Some **Safari** browsers may have issues with WebSocket connections when the window is unfocused

# <img src="/youtube.png" class="tab-icon"> 3 · Video

*Coming soon*