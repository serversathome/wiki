---
title: Note Discovery
description: A guide to deploying Note Discovery
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:06:46.282Z
---

# <img src="/notediscovery.png" class="tab-icon"> What is Note Discovery?
NoteDiscovery is a lightweight, self-hosted note-taking application that puts you in complete control of your knowledge base. Write, organize, and discover your notes with a beautiful, modern interface—all running on your own server.

<div class="glance">
  <div><span>Port</span><b><code>8000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/gamosoft/NoteDiscovery"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Note Discovery
1. Create the data folder in the TrueNAS shell as root:
    ```bash
    mkdir -p /mnt/tank/configs/notediscovery/data/_templates
    ```
1. *(Optional)* Copy the example note templates into the data folder:
    ```bash
    git clone --depth 1 https://github.com/gamosoft/NoteDiscovery /tmp/notediscovery-src
    cp /tmp/notediscovery-src/documentation/templates/*.md /mnt/tank/configs/notediscovery/data/_templates/
    rm -rf /tmp/notediscovery-src
    ```
1. Create a new stack in Dockge with this compose file:
    ```yaml
    services:
      notediscovery:
        image: ghcr.io/gamosoft/notediscovery:latest
        container_name: notediscovery
        ports:
          - "8000:8000"
        volumes:
          - /mnt/tank/configs/notediscovery/data:/app/data
        restart: unless-stopped
        environment:
          - TZ=America/New_York
        healthcheck:
          test: ["CMD", "python", "-c", "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')"]
          interval: 60s
          timeout: 3s
          retries: 3
          start_period: 5s
    ```
1. Start the stack and navigate to `http://IP:8000`

> Themes, plugins, locales and the default config are built into the prebuilt image, so you no longer need to clone the repo (the repo's own `docker-compose.yml` builds from source and is meant for development). To expose the built-in MCP server over HTTP, add `MCP_ENABLED=true` and point your client at `http://IP:8000/mcp`.
{.is-info}
