---
title: GPU Hot
description: A guide to deploying GPU Hot
published: true
date: 2026-01-15T15:29:24.364Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:58.467Z
---

# What is GPU Hot?
Real-time NVIDIA GPU monitoring dashboard. Web-based, no SSH required.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy GPU Hot

```yaml
services:
  gpu-hot:
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities:
                - gpu
    ports:
      - 1312:1312
    image: ghcr.io/psalias2006/gpu-hot:latest
    pid: host
```