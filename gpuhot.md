---
title: GPU Hot
description: A guide to deploying GPU Hot
published: true
date: 2025-11-04T21:48:09.183Z
tags: 
editor: markdown
dateCreated: 2025-11-04T21:48:09.183Z
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
```