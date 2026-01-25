---
title: Obsidian
description: A guide to installing containerized Obsidian
published: true
date: 2026-01-25T20:27:15.573Z
tags: productivity, docker
editor: markdown
dateCreated: 2026-01-25T18:18:10.690Z
---

# <img src="/obsidian.png" class="tab-icon"> What is Obsidian?

Obsidian is an application which facilitates note-taking, organization, relational thinking, and brainstorming with an ecosystem of available plugins to extend its utility even further. 

Many may elect to use a more "conventional" method of running Obsidian as a local application and then synchronizing the files by your method of choice (for example: via Syncthing) but it is also an option to just have it in a container. The default installation does not appear to implement any type of login, so exposing this container to the internet, without benefit of at least a secure vpn or the like, would be **highly** advised against.

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

Navigate to `http://{serverIP}:3001` or whichever port you designated and you will be presented with the Welcome screen for Obsidian.