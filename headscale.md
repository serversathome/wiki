---
title: Headscale
description: A guide to deploy Headscale with the Headscale-UI
published: true
date: 2025-08-10T12:47:16.513Z
tags: 
editor: markdown
dateCreated: 2025-03-24T10:59:55.365Z
---

# ![](/headscale.png){class="tab-icon"} What is Headscale?
Headscale aims to implement a self-hosted, open source alternative to the Tailscale control server. Headscale's goal is to provide self-hosters and hobbyists with an open-source server they can use for their projects and labs. It implements a narrow scope, a single Tailscale network (tailnet), suitable for a personal use, or a small open-source organisation.

# What is Headscale-Admin?
A web frontend for the headscale Tailscale-compatible coordination server.

# 1 · Prerequisites
- A Linux system with root access and a public IP address *(we recommend Ubuntu or Debian based systems)*
- [Docker](/Docker) installed on the server
- A domain name pointed to your server's IP address
- TCP ports 80 and 443 open

# 2 · Choosing a VPS
Headscale is best run from somewhere outside your network, ideally in the cloud. As such, you need to have a VPS to install Headscale.

A minimal VPS instance with 1 vCPU, 1GB RAM, and 8GB SSD will perform perfectly well for most use cases. In some cases, you may be able to get away with even less.

One option is [this option from Rack Nerd](https://my.racknerd.com/aff.php?aff=15328&pid=913) and honestly it's a great choice, but any VPS will do. *Note this is an affiliate link*

# 3 · Deploy Radarr
# {.tabset}
## <img src="/windows-terminal.png" class="tab-icon"> Script
The easiest way to deploy this is with my script, which (after you have met all the pre-requisites) will pull the docker containers, insert your FQDN, and generate an API key for you.

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
    image: 'headscale/headscale:latest'
    container_name: 'headscale'
    restart: 'unless-stopped'
    command: 'serve'
    volumes:
      - './data:/var/lib/headscale'
      - './configs/headscale:/etc/headscale'
    environment:
      TZ: 'America/New_York'
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.headscale.rule=Host(`my.domain.com`)"
      - "traefik.http.routers.headscale.tls.certresolver=myresolver"
      - "traefik.http.routers.headscale.entrypoints=websecure"
      - "traefik.http.routers.headscale.tls=true"
      - "traefik.http.services.headscale.loadbalancer.server.port=8080"

  headscale-admin:
    image: 'goodieshq/headscale-admin:latest'
    container_name: 'headscale-admin'
    restart: 'unless-stopped'
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.headscale-admin.rule=Host(`my.domain.com`) && PathPrefix(`/admin`)" # Fixed here
      - "traefik.http.routers.headscale-admin.entrypoints=websecure"
      - "traefik.http.routers.headscale-admin.tls=true"
      - "traefik.http.services.headscale-admin.loadbalancer.server.port=80"

  traefik:
    image: "traefik:latest"
    container_name: "traefik"
    command:
      - "--api.insecure=true"
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
    volumes:
      - "./letsencrypt:/letsencrypt"
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
```
- Replace `my.domain.com` with your FQDN (e.g. headscale.domain.com)

### Headscale Config Yaml
```yaml
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
    stun_listen_addr: "0.0.0.0:3478"
    private_key_path: /var/lib/headscale/derp_server_private.key
    automatically_add_embedded_derp_region: true
    ipv4: 1.2.3.4
    ipv6: 2001:db8::1
  urls:
    - https://controlplane.tailscale.com/derpmap/default
  paths: []
  auto_update_enabled: true
  update_frequency: 24h
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
  base_domain: example.com
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
```
- Replace `my.domain.com` with your FQDN (e.g. headscale.domain.com)

# 4 · Logging In
1. Run this command on the host to generate an API Key (this is not necessary if you ran the script):
    ```bash
    docker exec -it headscale headscale apikey create
    ```
1. Navigate to `https://your.domain.com/admin/settings`
1. Input your API key then refresh the page

# <img src="/youtube.png" class="tab-icon"> 5 · Video
[](https://youtu.be/r-qn6DrJ6IA)
