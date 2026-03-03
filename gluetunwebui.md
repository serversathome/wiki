---
title: Gluetun-webui
description: A guide to deploying Gluetun-webui
published: true
date: 2026-03-03T17:52:24.390Z
tags: 
editor: markdown
dateCreated: 2026-03-03T17:52:24.390Z
---

# What is Gluetun WebUI?

**Gluetun WebUI** is a lightweight web interface for monitoring and controlling [Gluetun](https://github.com/qdm12/gluetun) — the popular VPN client container for Docker. It provides a clean dashboard showing your VPN connection status, public IP details, port forwarding info, DNS status, and gives you start/stop controls right from the browser. It works with both WireGuard and OpenVPN without any configuration changes.


> 
> Gluetun WebUI requires a running [Gluetun](https://github.com/qdm12/gluetun) container with its HTTP control server enabled (default port `8000`). Both containers must be on the same Docker network.
{.is-info}

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Gluetun WebUI

```yaml
services:
  gluetun-webui:
    image: scuzza/gluetun-webui:latest
    container_name: gluetun-webui
    ports:
      - "3000:3000"
    environment:
      - GLUETUN_CONTROL_URL=http://gluetun:8000
      # Uncomment ONE of the auth options below if Gluetun has auth enabled:
      # Bearer token (HTTP_CONTROL_SERVER_AUTH=apikey:yourtoken in gluetun)
      #- GLUETUN_API_KEY=yourtoken
      # HTTP Basic auth (HTTP_CONTROL_SERVER_AUTH=username:password in gluetun)
      #- GLUETUN_USER=username
      #- GLUETUN_PASSWORD=password
    networks:
      - your_network_name
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
2. If your Gluetun instance has authentication enabled, uncomment the appropriate environment variables (`GLUETUN_API_KEY` for bearer token auth, or `GLUETUN_USER` / `GLUETUN_PASSWORD` for HTTP basic auth)
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
{.dense}

## 2.2 Authentication

Gluetun WebUI supports the same authentication methods as Gluetun's control server:

- **Bearer Token Auth** — Set `GLUETUN_API_KEY` to match the token configured in Gluetun via `HTTP_CONTROL_SERVER_AUTH=apikey:yourtoken`
- **HTTP Basic Auth** — Set `GLUETUN_USER` and `GLUETUN_PASSWORD` to match the credentials configured in Gluetun via `HTTP_CONTROL_SERVER_AUTH=username:password`

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

