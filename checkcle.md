---
title: Checkcle
description: A guide to deploying Checkle
published: true
date: 2025-12-10T20:02:07.332Z
tags: 
editor: markdown
dateCreated: 2025-12-10T19:51:54.490Z
---

# <img src="/checkcle.png" class="tab-icon"> What is Checkcle?

CheckCle is a self-hosted, open-source monitoring platform for seamless, real-time full-stack systems, applications, and infrastructure. It provides real-time uptime monitoring, distributed checks, incident tracking, and alerts. All deployable anywhere.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Checkcle
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
1. Navigate to `http://IP:8090`
1. The default email is `admin@example.com` and the default password is `Admin123456`
