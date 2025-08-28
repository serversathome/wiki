---
title: Dockpeek
description: A guide to deploying Dockpeek
published: true
date: 2025-08-28T17:58:03.029Z
tags: 
editor: markdown
dateCreated: 2025-08-28T17:58:03.029Z
---

# <img src="/dockpeek.png" class="tab-icon"> What is Dockpeek?
Dockpeek is a lightweight, self-hosted Docker dashboard that allows you to view and access exposed container ports with a clean, click-to-access interface. It supports both local Docker sockets and remote hosts via socket-proxy, making it easy to manage multiple Docker environments from a single place. Additionally, Dockpeek includes built-in image update checking, so you can easily see if newer versions of your container images are available.

- Port Mapping Overview – Quickly see all running containers and their exposed ports.
- Click-to-Access URLs – Open containerized web apps instantly with a single click.
- Multi-Host Support – Manage multiple Docker hosts and sockets within one dashboard.
- Zero Configuration – Automatically detects running containers with no setup required.
- Image Update Checking – Monitor available updates for your container images.


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Dockpeek
```yaml
services:
  dockpeek:
    image: ghcr.io/dockpeek/dockpeek:latest
    container_name: dockpeek
    environment:
      - SECRET_KEY=my_secret_key   # Set secret key
      - USERNAME=admin             # Change default username
      - PASSWORD=admin             # Change default password
    ports:
      - "3420:8000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    restart: unless-stopped
```