---
title: GPU Hot
description: A guide to deploying GPU Hot
published: true
date: '2026-11-04T16:23:31.822Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:04:58.467Z'
---

# What is GPU Hot?
Real-time NVIDIA GPU monitoring dashboard. Web-based, no SSH required.

# :logos:docker-icon: 1 · Deploy GPU Hot

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
    pid: host # lets the dashboard show process names
    init: true
    restart: unless-stopped
    # environment:
    #   - NVIDIA_SMI=true      # force nvidia-smi mode if metrics don't appear on older GPUs
    #   - UPDATE_INTERVAL=0.5  # polling interval in seconds; raise it if performance suffers
```

> [!NOTE]
> The host needs the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) installed for the GPU reservation to work.

> [!NOTE]
> **Monitoring several GPU servers?** Run GPU Hot on each one (optionally with `NODE_NAME` set), then run one more instance in hub mode, which needs no GPU, with `GPU_HOT_MODE=hub` and `NODE_URLS=http://server1:1312,http://server2:1312`. The hub shows all nodes on a single dashboard.
