---
title: Headscale
description: A guide to deploy Headscale with the Headscale-UI
published: true
date: 2026-05-13T11:55:47.891Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:05:03.838Z
---

# ![](/headscale.png){class="tab-icon"} What is Headscale?
Headscale aims to implement a self-hosted, open source alternative to the Tailscale control server. Headscale's goal is to provide self-hosters and hobbyists with an open-source server they can use for their projects and labs. It implements a narrow scope, a single Tailscale network (tailnet), suitable for a personal use, or a small open-source organisation.

# What is Headscale-UI?
A web frontend for the headscale Tailscale-compatible coordination server by [gurucomputing](https://github.com/gurucomputing/headscale-ui). It provides a simple browser-based interface for managing users, devices, pre-auth keys, and routes. The UI stores your API key locally in the browser — no backend configuration needed.

# 1 · Prerequisites
- A Linux system with root access and a public IP address *(we recommend Ubuntu or Debian based systems)*
- [Docker](/Docker) installed on the server
- A domain name pointed to your server's IP address
- TCP ports 80 and 443 open
- UDP port 3478 open *(for STUN/DERP NAT traversal)*

# 2 · Choosing a VPS
Headscale is best run from somewhere outside your network, ideally in the cloud. As such, you need to have a VPS to install Headscale.

A minimal VPS instance with 1 vCPU, 1GB RAM, and 8GB SSD will perform perfectly well for most use cases. In some cases, you may be able to get away with even less.

One option is [Racknerd](/racknerd) and honestly it's a great choice, but any VPS will do. *Note this is an affiliate link*

> Check out my guide for [Racknerd](/racknerd)
{.is-info}


# 3 · Deploy Headscale
# {.tabset}
## <img src="/windows-terminal.png" class="tab-icon"> Script
The easiest way to deploy this is with my script, which (after you have met all the pre-requisites) will pull the docker containers, insert your FQDN, and generate an API key for you.

The script will prompt you for:
- Your full domain (e.g. `headscale.example.com`)
- Your email address for Let's Encrypt certificates
- Your VPS public IPv4 address (for the embedded DERP relay)
- A MagicDNS base domain (must be different from your server domain, e.g. `ts.example.com`)

Run this command to launch the script:
```bash
wget -q -O headscale.sh https://raw.githubusercontent.com/serversathome/ServersatHome/main/headscale.sh && chmod +x headscale.sh && sudo ./headscale.sh
```

> Follow the directions at the end of the script for login information
{.is-info}


## <img src="/docker.png" class="tab-icon"> Docker Compose
### Headscale Container
```yaml
services:
  headscale:
    image: 'headscale/headscale:0.28.0'
    container_name: 'headscale'
    restart: 'unless-stopped'
    command: 'serve'
    read_only: true
    tmpfs:
      - /var/run/headscale
    volumes:
      - './config:/etc/headscale:ro'
      - './lib:/var/lib/headscale'
    environment:
      TZ: 'America/New_York'
    healthcheck:
      test: ["CMD", "headscale", "health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 15s
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.headscale.rule=Host(`my.domain.com`)"
      - "traefik.http.routers.headscale.tls.certresolver=myresolver"
      - "traefik.http.routers.headscale.entrypoints=websecure"
      - "traefik.http.routers.headscale.tls=true"
      - "traefik.http.services.headscale.loadbalancer.server.port=8080"

  headscale-ui:
    image: 'ghcr.io/gurucomputing/headscale-ui:2025.08.23'
    container_name: 'headscale-ui'
    restart: 'unless-stopped'
    labels:
      - "traefik.enable=true"
      - "traefik.http.services.headscale-ui.loadbalancer.server.port=8080"
      - "traefik.http.routers.headscale-ui.rule=Host(`my.domain.com`) && PathPrefix(`/web`)"
      - "traefik.http.routers.headscale-ui.entrypoints=websecure"
      - "traefik.http.routers.headscale-ui.tls=true"
      - "traefik.http.routers.headscale-ui.tls.certresolver=myresolver"

  traefik:
    image: "traefik:v3.3"
    container_name: "traefik"
    restart: "unless-stopped"
    command:
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.web.http.redirections.entrypoint.to=websecure"
      - "--entrypoints.web.http.redirections.entrypoint.scheme=https"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.myresolver.acme.httpchallenge=true"
      - "--certificatesresolvers.myresolver.acme.httpchallenge.entrypoint=web"
      - "--certificatesresolvers.myresolver.acme.email=you@yourdomain.com"
      - "--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json"
    ports:
      - "80:80"
      - "443:443"
      - "3478:3478/udp"
    volumes:
      - "./letsencrypt:/letsencrypt"
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
```
- Replace `my.domain.com` with your FQDN (e.g. headscale.domain.com)
- Replace `you@yourdomain.com` with your actual email for Let's Encrypt

> Headscale-UI serves on port **8080** and uses the `/web` path. This is different from the old headscale-admin which used port 80 and the `/admin` path.
{.is-warning}

### Headscale Config Yaml
Save this as `config/config.yaml`:
```yaml
---
server_url: https://my.domain.com
listen_addr: 0.0.0.0:8080
metrics_listen_addr: 127.0.0.1:9090
grpc_listen_addr: 127.0.0.1:50443
grpc_allow_insecure: false

noise:
  private_key_path: /var/lib/headscale/noise_private.key

prefixes:
  v4: 100.64.0.0/10
  v6: fd7a:115c:a1e0::/48
  allocation: sequential

derp:
  server:
    enabled: true
    region_id: 999
    region_code: "headscale"
    region_name: "Headscale Embedded DERP"
    verify_clients: true
    stun_listen_addr: "0.0.0.0:3478"
    private_key_path: /var/lib/headscale/derp_server_private.key
    automatically_add_embedded_derp_region: true
    ipv4: 1.2.3.4
  urls:
    - https://controlplane.tailscale.com/derpmap/default
  paths: []
  auto_update_enabled: true
  update_frequency: 3h

disable_check_updates: false
ephemeral_node_inactivity_timeout: 30m

database:
  type: sqlite
  debug: false
  gorm:
    prepare_stmt: true
    parameterized_queries: true
    skip_err_record_not_found: true
    slow_threshold: 1000
  sqlite:
    path: /var/lib/headscale/db.sqlite
    write_ahead_log: true
    wal_autocheckpoint: 1000

acme_url: https://acme-v02.api.letsencrypt.org/directory
acme_email: ""
tls_letsencrypt_hostname: ""
tls_letsencrypt_cache_dir: /var/lib/headscale/cache
tls_letsencrypt_challenge_type: HTTP-01
tls_letsencrypt_listen: ":http"
tls_cert_path: ""
tls_key_path: ""

log:
  format: text
  level: info

policy:
  mode: database
  path: ""

dns:
  magic_dns: true
  base_domain: ts.example.com
  override_local_dns: true
  nameservers:
    global:
      - 1.1.1.1
      - 1.0.0.1
      - 2606:4700:4700::1111
      - 2606:4700:4700::1001
    split: {}
  search_domains: []
  extra_records: []

unix_socket: /var/run/headscale/headscale.sock
unix_socket_permission: "0770"

logtail:
  enabled: false

randomize_client_port: false

taildrop:
  enabled: true
```
- Replace `my.domain.com` with your FQDN (e.g. headscale.domain.com)
- Replace `1.2.3.4` with your VPS public IPv4 address
- Replace `ts.example.com` with your MagicDNS base domain (this **must** be different from your server domain)

> The `dns.base_domain` cannot be the same as your `server_url` domain. Use a subdomain like `ts.yourdomain.com` or a completely different domain.
{.is-warning}

# 4 · Logging In
1. Run this command on the host to generate an API Key (this is not necessary if you ran the script):
    ```bash
    docker exec headscale headscale apikeys create
    ```
1. Navigate to `https://your.domain.com/web`
1. Click **Settings** in the Headscale-UI
1. Set the **Headscale URL** to `https://your.domain.com`
1. Paste your **API Key** and save
1. The UI should now populate with your headscale data

# 5 · Connecting Clients
To connect a device to your headscale instance:
```bash
tailscale up --login-server=https://your.domain.com
```

> On first connection, you'll need to create a user and register the node. Run `docker exec headscale headscale users create myuser` to create a user, then follow the registration URL shown in the Tailscale client output.
{.is-info}

# 6 · Useful Commands
| Command | Description |
|---------|-------------|
| `docker exec headscale headscale users create <name>` | Create a new user |
| `docker exec headscale headscale users list` | List all users |
| `docker exec headscale headscale nodes list` | List all connected nodes |
| `docker exec headscale headscale apikeys create` | Generate a new API key |
| `docker exec headscale headscale apikeys list` | List existing API keys |
| `docker exec headscale headscale preauthkeys create --user <name>` | Create a pre-auth key |
{.dense}

# <img src="/youtube.png" class="tab-icon"> 7 · Video
https://youtu.be/r-qn6DrJ6IA