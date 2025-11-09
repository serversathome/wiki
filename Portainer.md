---
title: Portainer
description: A guide to installing Portainer on Ubuntu Server LTS
published: true
date: 2025-11-09T14:17:07.496Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:41:40.846Z
---

# <img src="/portainer2.png" class="tab-icon"> What is Portainer?

Portainer is a popular Docker UI that helps you visualize your containers, images, volumes and networks. Portainer helps you take control of the Docker resources on your machine, avoiding lengthy terminal commands.

# Installation
# {.tabset}
## <img src="/truenas.png" class="tab-icon"> TrueNAS

![](/screenshot_from_2024-11-08_11-29-50.png)

1. Change **Portainer Data Storage** to **Host Path**. 
1. Change the **Port** to 9443 since that is the default for Portainer and is referenced in all their documentation.

## <img src="/docker.png" class="tab-icon"> Docker

> Note that Docker has to be installed before you can follow these steps!
{.is-warning}


### Docker Run

Run all of these commands in sequence:
```bash
docker volume create portainer_data
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

### Docker Compose

```yaml
services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: unless-stopped
    ports:
      - 8000:8000
      - 9443:9443
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /mnt/tank/configs/portainer:/data
```

# Login

1. Portainer is now running on port 9443, behind https. To login, enter `https://{serverIP}:9443` in the address bar, then create your user and password. 

1. After logging in, click the box for **Get Started**. 

1. To deploy Docker Compose files, go to **Stacks** > **\+ Add Stack** and copy & paste the compose.yaml there.

# Video
[](https://www.youtube.com/watch?v=wsixvmNPrlU)