---
title: Pingora Proxy Manager
description: A guide to deploying Pingora Proxy Manager
published: true
date: 2026-01-15T15:30:51.361Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:19.270Z
---

# <img src="/pingora-proxy-manager.png" class="tab-icon"> What is Pingora Proxy Manager?
A high-performance, zero-downtime reverse proxy manager built on Cloudflare's Pingora.

Simple, Modern, and Fast. Now supports Wildcard SSL & TCP/UDP Streams!

⚡️ High Performance: Built on Rust & Pingora, capable of handling high traffic with low latency.
🔄 Zero-Downtime Configuration: Dynamic reconfiguration without restarting the process.
🔒 SSL/TLS Automation
🌐 Proxy Hosts: Easy management of virtual hosts, locations, and path rewriting.
📡 Streams (L4): TCP and UDP forwarding for databases, game servers, etc.
🛡️ Access Control: IP whitelisting/blacklisting and Basic Authentication support.
🎨 Modern Dashboard: Clean and responsive UI built with React, Tailwind CSS, and shadcn/ui.
🐳 Docker Ready: Single container deployment for easy setup and maintenance.

<div class="glance">
  <div><span>Port</span><b><code>81</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
</div>


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Pingora Proxy Manager
```yaml
services:
  pingora-proxy:
    image: dduldduck/pingora-proxy-manager:latest
    container_name: pingora-proxy
    restart: unless-stopped
    ports:
      - "80:8080"   # HTTP Proxy (Backend listens on 8080)
      - "81:81"     # Dashboard/API (Backend listens on 81)
      # Map 443 if you want to serve HTTPS directly (requires privilege or capability)
      # - "443:443" 
    volumes:
      - /mnt/tank/configs/pingora/data:/app/data
      - /mnt/tank/configs/pingora:/app/logs
    environment:
      - JWT_SECRET=changeme_in_production_please
      - RUST_LOG=info
```

# 2 · Logging In
1. Navigate to `http://IP:81` to get to the dashboard
1. The default user is `admin` and the password is `changeme`