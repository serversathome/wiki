---
title: Obsidian
description: A guide to installing containerized Obsidian
published: true
date: 2026-10-28T12:00:00.000Z
tags: productivity, docker
editor: markdown
dateCreated: 2026-01-25T18:18:10.690Z
---

# <img src="/obsidian.png" class="tab-icon"> What is Obsidian?

Obsidian is an application which facilitates note-taking, organization, relational thinking, and brainstorming with an ecosystem of available plugins to extend its utility even further. 

Many may elect to use a more "conventional" method of running Obsidian as a local application and then synchronizing the files by your method of choice (for example: via Syncthing) but it is also an option to just have it in a container. The default installation has no login unless you set the optional `CUSTOM_USER` and `PASSWORD` variables, which enable HTTP basic auth. Even then, that login is only meant to keep the kids out, so exposing this container to the internet, without benefit of at least a secure vpn or the like, would be **highly** advised against.

<div class="glance">
  <div><span>Port</span><b><code>3001</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/linuxserver/docker-obsidian"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Obsidian


```yaml
services:
  obsidian:
    image: lscr.io/linuxserver/obsidian:latest
    container_name: obsidian
    environment:
      - PUID=568
      - PGID=568
      - TZ=Etc/UTC
      - CUSTOM_USER=admin #optional, enables basic auth
      - PASSWORD=<strong password> #optional, enables basic auth
    volumes:
      - /mnt/tank/configs/obsidian:/config
      - /mnt/tank/vaults:/vaults #optional
    ports:
      - 3001:3001
    shm_size: 1gb
    restart: unless-stopped
```

- Change the `TZ` to your local timezone if you wish
- Change the `volumes` path for `configs` to reflect your desired location for the container's configuration files
- I have opted to include an additional bind to a location specifically where I want my Vaults to go, separately from the configuration files. If you would like to do the same, the second line under `volumes` can point to that location on your server and then the corresponding path within the container `/vaults`.



# 2 · Accessing Obsidian

Navigate to `https://{serverIP}:3001` or whichever port you designated. HTTPS is required since the image moved to Selkies (July 2025), and plain `http://` on 3001 will not load. Your browser will warn about the self-signed certificate; accept it and you will be presented with the Welcome screen for Obsidian.

> If the desktop doesn't render on older hardware or GPUs, add `- PIXELFLUX_WAYLAND=false` to the environment to turn off the default Wayland mode.
{.is-info}