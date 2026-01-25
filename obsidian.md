---
title: Obsidian
description: A guide to installing containerized Obsidian
published: true
date: 2026-01-25T18:18:10.690Z
tags: productivity, docker
editor: markdown
dateCreated: 2026-01-25T18:18:10.690Z
---

# What is Obsidian?

Obsidian is an application which facilitates note-taking, organization, relational thinking, and brainstorming with an ecosystem of available plugins to extend its utility even further. 

Many may elect to use a more "conventional" method of running Obsidian as a local application and then synchronizing the files by your method of choice (for example: via Syncthing) but it is also an option to just have it in a container. The default installation does not appear to implement any type of login, so exposing this container to the internet, without benefit of at least a secure vpn or the like, would be **highly** advised against.

# 1 · Deploy Obsidian
# {.tabset}
## <img src="/dockge.png" class="tab-icon"> Dockge

### Compose Configuration

1. Select "+ Compose" in Dockge.
2. Clear out the Compose field on the right and paste in the provided Compose file seen below.
3. Make sure the dataset has permissions set for the **apps** user - it is already set here with ``568``. 
4. Change the ``TZ`` to your local timezone if you wish.
5. Change the ``volumes`` path for ``configs`` to reflect your desired location for the container's configuration files.
6. **Optional** By default, the above mentioned ``configs`` folder will also be the default, and only, location for your Obsidian Vaults. 
I have opted to include an additional bind to a location specifically where I want my Vaults to go, separately from the configuration files. If you would like to do the same, the second line under ``volumes`` can point to that location on your server and then the corresponding path within the container ``/vaults``.
7. If you need to change the port by which Obsidian is accessed, you can do that under ``ports``. ``3001`` is the default port and uses HTTPS.

### Docker Compose

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

## <img src="/truenas.png" class="tab-icon"> TrueNAS via YAML


1. In the TrueNAS Apps tab, select Discover Apps.
2. Next to the Custom App button, select the three dogs and click "Install via YAML"
3. Give the container a name and paste in the Compose file from the Dockge tab in this wiki entry, noting the outlined configuration options.
4. Click the Save button and the container will Deploy.


# 2 · Accessing Obsidian

Navigate to http://{serverIP}:3001 or whichever port you designated and you will be presented with the Welcome screen for Obsidian.