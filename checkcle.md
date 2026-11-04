---
title: Checkcle
description: A guide to deploying Checkle
published: true
date: '2026-11-04T23:31:50.195Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:03:37.230Z'
---

# What is Checkcle?

CheckCle is a self-hosted, open-source monitoring platform for seamless, real-time full-stack systems, applications, and infrastructure. It provides real-time uptime monitoring, distributed checks, incident tracking, and alerts. All deployable anywhere.

# :logos:docker-icon: 1 · Deploy Checkcle
```yaml
services:
  checkcle:
    image: operacle/checkcle:latest
    container_name: checkcle
    restart: unless-stopped
    ports:
      - 8090:8090
    volumes:
      - /mnt/tank/configs/checkcle:/mnt/pb_data  # Host directory mapped to container path
```

# Logging In

::block-steps
1. Navigate to `http://IP:8090`
1. The default email is `admin@example.com` and the default password is `Admin123456`
1. Change the default password right after your first login
::

> [!WARNING]
> Anyone who can reach port 8090 can log in with the default credentials until you change them.

