---
title: Gluetun-webui
description: A guide to deploying Gluetun-webui
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-03-03T17:52:24.390Z
---

# What is Gluetun WebUI?

**Gluetun WebUI** is a lightweight web interface for monitoring and controlling [Gluetun](https://github.com/qdm12/gluetun) — the popular VPN client container for Docker. It provides a clean dashboard showing your VPN connection status, public IP details, port forwarding info, DNS status, and gives you start/stop controls right from the browser. It works with both WireGuard and OpenVPN without any configuration changes.


> 
> Gluetun WebUI requires a running [Gluetun](https://github.com/qdm12/gluetun) container with its HTTP control server enabled (default port `8000`). Both containers must be on the same Docker network. Current Gluetun versions **require** authentication on every control server route, so you will need to set up an API key or basic auth in Gluetun (see 2.2).
{.is-info}

<div class="glance">
  <div><span>Port</span><b><code>3000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/Sir-Scuzza/gluetun-webui"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Gluetun WebUI

```yaml
services:
  gluetun-webui:
    image: scuzza/gluetun-webui:latest
    container_name: gluetun-webui
    ports:
      - "3000:3000" # use "127.0.0.1:3000:3000" if you only reach it through a reverse proxy
    environment:
      - GLUETUN_CONTROL_URL=http://gluetun:8000
      # Gluetun requires auth. Uncomment ONE option to match your Gluetun config:
      # API key (Gluetun: HTTP_CONTROL_SERVER_AUTH_DEFAULT_ROLE='{"auth":"apikey","apikey":"yourtoken"}')
      #- GLUETUN_API_KEY=yourtoken
      # HTTP Basic auth (Gluetun: HTTP_CONTROL_SERVER_AUTH_DEFAULT_ROLE='{"auth":"basic","username":"username","password":"password"}')
      #- GLUETUN_USER=username
      #- GLUETUN_PASSWORD=password
    networks:
      - ext-network
    restart: unless-stopped
    read_only: true
    tmpfs:
      - /tmp
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:3000/api/health"]
      interval: 30s
      timeout: 5s
      start_period: 10s
      retries: 3

networks:
  ext-network:
    external: true
    name: your_network_name
```

1. Replace `your_network_name` with the Docker network your Gluetun container is running on
2. Uncomment the auth variables that match your Gluetun control server config (`GLUETUN_API_KEY` for API key auth, or `GLUETUN_USER` / `GLUETUN_PASSWORD` for HTTP basic auth). Without them the UI gets `401` errors from Gluetun.
3. Deploy the stack and access the UI at `http://your-server-ip:3000`


# 2 · Configuration

## 2.1 Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `3000` | Port the web UI listens on |
| `GLUETUN_CONTROL_URL` | `http://gluetun:8000` | URL of Gluetun's HTTP control server |
| `GLUETUN_API_KEY` | *(empty)* | Bearer token if Gluetun API key auth is enabled |
| `GLUETUN_USER` | *(empty)* | Username for HTTP Basic auth |
| `GLUETUN_PASSWORD` | *(empty)* | Password for HTTP Basic auth |
| `GLUETUN_{N}_URL` / `GLUETUN_{N}_NAME` | *(empty)* / `Instance {N}` | Monitor multiple Gluetun instances (N = 1–20). Each shows as its own dashboard |
| `GLUETUN_{N}_API_KEY` / `GLUETUN_{N}_USER` / `GLUETUN_{N}_PASSWORD` | *(empty)* | Per-instance auth |
{.dense}

For several VPNs, replace `GLUETUN_CONTROL_URL` with numbered variables:

```yaml
    environment:
      - GLUETUN_1_NAME=VPN - London
      - GLUETUN_1_URL=http://gluetun-1:8000
      - GLUETUN_1_API_KEY=token1
      - GLUETUN_2_NAME=VPN - Amsterdam
      - GLUETUN_2_URL=http://gluetun-2:8000
      - GLUETUN_2_API_KEY=token2
```

## 2.2 Authentication

All of Gluetun's control server routes are private by default and **require** authentication. Configure it on the **Gluetun** container, either with the `HTTP_CONTROL_SERVER_AUTH_DEFAULT_ROLE` environment variable or with a `config.toml` mounted at `/gluetun/auth/config.toml` (see the [Gluetun control server docs](https://github.com/qdm12/gluetun-wiki/blob/main/setup/advanced/control-server.md)). Then give Gluetun WebUI the matching credentials:

- **API Key Auth** — In Gluetun set `HTTP_CONTROL_SERVER_AUTH_DEFAULT_ROLE='{"auth":"apikey","apikey":"yourtoken"}'`, then set `GLUETUN_API_KEY=yourtoken` here. Generate a key with `docker run --rm qmcgaw/gluetun genkey`.
- **HTTP Basic Auth** — In Gluetun set `HTTP_CONTROL_SERVER_AUTH_DEFAULT_ROLE='{"auth":"basic","username":"username","password":"password"}'`, then set `GLUETUN_USER` and `GLUETUN_PASSWORD` here.

> 
> If you expose this service beyond localhost, place it behind a reverse proxy with authentication (Nginx Proxy Manager, Caddy, Traefik, etc.). The VPN start/stop endpoints have no UI-layer authentication by default.
{.is-danger}

## 2.3 Dashboard Overview

The dashboard displays several status sections:

- **VPN Status Banner** — Green (connected), Yellow (paused), Red (disconnected), or Yellow (unknown/unreachable)
- **Public IP Card** — Shows your exit IP address, country, region, city, and organisation
- **VPN Details Card** — Provider, protocol (WireGuard/OpenVPN auto-detected), server hostname, country, city
- **Port Forwarding** — Displays the currently forwarded port if enabled in Gluetun
- **DNS Status** — Confirms Gluetun's internal DNS resolver is running
- **Poll History** — Last 30 status checks displayed as color-coded ticks

## 2.4 Auto-Refresh

The UI supports configurable polling intervals. Use the dropdown in the interface to select your preferred refresh rate: 5 seconds, 10 seconds, 30 seconds, 60 seconds, or off. The polling system uses recursive `setTimeout` with an in-flight guard to prevent overlapping requests.

