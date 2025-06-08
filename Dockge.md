---
title: Dockge
description: A guide to installing Dockge on Ubuntu Server LTS
published: true
date: 2025-06-08T18:39:30.907Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:43:09.366Z
---

![](/dockge.png)

![](https://wiki.hydrology.cc/dockgedash.png)

# What is Dockge?

Dockge is a self-hosted Docker stack manager developed by the same person behind the popular software Uptime Kuma.

This software allows you to manage multiple Docker compose files from a single, easy-to-use interface. It is similar to the stack system Portainer implements but cleaner and simpler to use.

# Installation
# {.tabset}
## TrueNAS

![](/screenshot_from_2024-11-08_11-34-31.png)

1. Change the **WebUI Port** to **5001** since that is the default normally used by Dockge.
1. Change the **Dockge Stacks Storage** to **Hostpath** and make sure the dataset has permissions set for the **apps** user. 

## Docker

> Note that Docker has to be installed before you can follow these steps!
{.is-warning}


### Docker Run

```bash
# Create directories that store your stacks and stores Dockge's stack
mkdir -p /opt/stacks /opt/dockge
cd /opt/dockge

# Download the compose.yaml
curl https://raw.githubusercontent.com/louislam/dockge/master/compose.yaml --output compose.yaml

# Start the server
docker compose up -d

# If you are using docker-compose V1 or Podman
# docker-compose up -d
```

### Docker Compose

```yaml
services:
 dockge:
   image: louislam/dockge:latest
   restart: unless-stopped
   container_name: dockge
   ports:
     - 5001:5001
   volumes:
     - /var/run/docker.sock:/var/run/docker.sock
     - /mnt/tank/configs/dockge:/app/data

     # If you want to use private registries, you need to share the auth file with Dockge:
     # - /root/.docker/:/root/.docker

     # Stacks Directory
     # ⚠️ READ IT CAREFULLY. If you did it wrong, your data could end up writing into a WRONG PATH.
     # ⚠️ 1. FULL path only. No relative path (MUST)
     # ⚠️ 2. Left Stacks Path === Right Stacks Path (MUST)
     - /opt/stacks:/opt/stacks
   environment:
     # Tell Dockge where is your stacks directory
     - DOCKGE_STACKS_DIR=/opt/stacks
```

# Logging In

Navigate to http://{serverIP}:5001 and create a user and password to login.

# YouTube Walkthrough

[https://youtu.be/LpAxsO7zAMA](https://youtu.be/mVbi6xkM-rk)