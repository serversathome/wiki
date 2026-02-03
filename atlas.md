---
title: Atlas
description: A guide to deploying Atlas
published: true
date: 2026-02-03T13:49:34.086Z
tags: 
editor: markdown
dateCreated: 2026-02-03T13:49:34.086Z
---

# <img src="/atlas-network.png" class="tab-icon"> What is Atlas?

**Atlas** is an open-source network infrastructure visualizer that scans, analyzes, and visualizes your network in real-time. Built with Go, FastAPI, and React, it automatically discovers Docker containers, local hosts, and neighboring devices on your subnet, then presents everything in an interactive dashboard.

**Key Features:**
- Scans Docker containers for IPs, MACs, ports, and network info
- Discovers hosts on your local subnet via ARP/Nmap
- Multi-interface scanning with support for multiple subnets
- Real-time interactive network graph visualization
- Deep port scanning with OS fingerprinting
- External IP discovery
- Scheduled auto-scans with configurable intervals
- Mobile-friendly responsive UI

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Atlas

```yaml
services:
  atlas:
    image: keinstien/atlas:latest
    container_name: atlas
    network_mode: host
    cap_add:
      - NET_RAW
      - NET_ADMIN
    environment:
      - ATLAS_UI_PORT=8888
      - ATLAS_API_PORT=8889
      - FASTSCAN_INTERVAL=3600
      - DOCKERSCAN_INTERVAL=3600
      - DEEPSCAN_INTERVAL=7200
      # - SCAN_SUBNETS=192.168.1.0/24,10.0.0.0/24
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

1. Deploy the stack
2. Navigate to `http://your-server-ip:8888`
3. Atlas will automatically begin scanning your network

> 
> Atlas requires `network_mode: host` and elevated capabilities (`NET_RAW`, `NET_ADMIN`) to perform network scanning. The Docker socket mount is needed to discover containers.
{.is-info}

# 2 · Configuration

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `ATLAS_UI_PORT` | Port for the web UI (Nginx) | 8888 |
| `ATLAS_API_PORT` | Port for the FastAPI backend | 8889 |
| `FASTSCAN_INTERVAL` | Seconds between fast ARP scans | 3600 |
| `DOCKERSCAN_INTERVAL` | Seconds between Docker container scans | 3600 |
| `DEEPSCAN_INTERVAL` | Seconds between deep port scans | 7200 |
| `SCAN_SUBNETS` | Comma-separated subnets to scan (auto-detects if not set) | Auto |
{.dense}

## Scanning Multiple Subnets

To scan multiple networks (e.g., your LAN and a remote server network), set the `SCAN_SUBNETS` variable:

```yaml
environment:
  - SCAN_SUBNETS=192.168.1.0/24,10.0.0.0/24,172.16.0.0/24
```

If not set, Atlas will auto-detect and scan the local subnet.

## Adjusting Scan Intervals

You can adjust scan intervals in three ways:
- Set environment variables at deployment (shown above)
- Change intervals dynamically through the Scripts Panel in the UI
- Use the Scheduler API endpoints

# 3 · Resources

- **GitHub:** https://github.com/karam-ajaj/atlas
- **Live Demo:** https://atlasdemo.vnerd.nl/

# <img src="/youtube.png" class="tab-icon"> 4 · Video

*No video yet — check back soon!*