---
title: Beszel
description: A guide to deploying Beszel
published: true
date: 2026-01-15T15:28:14.116Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:03:24.164Z
---

# <img src="/beszel.png" class="tab-icon"> What is Beszel?
Beszel is a lightweight server monitoring platform that includes Docker statistics, historical data, and alert functions.

It has a friendly web interface, simple configuration, and is ready to use out of the box. It supports automatic backup, multi-user, OAuth authentication, and API access.


# 1 · Deploy Beszel
# {.tabset}
## <img src="/truenas.png" class="tab-icon"> Server
```yaml
services:
  beszel:
    image: henrygd/beszel
    container_name: beszel
    restart: unless-stopped
    ports:
      - 8090:8090
    volumes:
      - /mnt/tank/configs/beszel:/beszel_data
```

## <img src="/linux.png" class="tab-icon"> Linux Client
```yaml
services:
  beszel-agent:
    image: henrygd/beszel-agent
    container_name: beszel-agent
    restart: unless-stopped
    network_mode: host
    volumes:
      - ./beszel_agent_data:/var/lib/beszel-agent
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      LISTEN: 45876
      KEY: "<public key>"
      HUB_URL: "http://localhost:8090"
      TOKEN: "<token>"
```

## <img src="/microsoft-windows.png" class="tab-icon"> Windows Client
To install beszel-agent use the following command:
```powershell
& iwr -useb https://get.beszel.dev -OutFile "$env:TEMP\install-agent.ps1"; & Powershell -ExecutionPolicy Bypass -File "$env:TEMP\install-agent.ps1"
```
To edit the service, see [these instructions from Beszel](https://beszel.dev/guide/agent-installation#edit-configuration).