---
title: Beszel
description: A guide to deploying Beszel
published: true
date: '2026-11-04T16:16:53.944Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:03:24.164Z'
---

# What is Beszel?
Beszel is a lightweight server monitoring platform that includes Docker statistics, historical data, and alert functions.

It has a friendly web interface, simple configuration, and is ready to use out of the box. It supports automatic backup, multi-user, OAuth authentication, and API access.

# 1 · Deploy Beszel
:::block-tabs
::block-tab{label="Server" icon="simple-icons:truenas"}
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

::

::block-tab{label="Linux Client" icon="simple-icons:linux"}
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

::

::block-tab{label="Windows Client" icon="simple-icons:windows"}
To install beszel-agent use the following command:
```powershell
& iwr -useb https://get.beszel.dev -OutFile "$env:TEMP\install-agent.ps1"; & Powershell -ExecutionPolicy Bypass -File "$env:TEMP\install-agent.ps1"
```
To edit the service, see [these instructions from Beszel](https://beszel.dev/guide/agent-installation#edit-configuration).

::

:::
