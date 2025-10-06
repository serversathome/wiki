---
title: Portracker
description: A guide to deploying Portracker via docker
published: true
date: 2025-10-06T23:21:58.251Z
tags: 
editor: markdown
dateCreated: 2025-08-04T12:30:09.026Z
---

# ![](/portracker.png){class="tab-icon"} What is Portracker?
By auto-discovering services on your systems, portracker provides a live, accurate map of your network. It helps eliminate manual tracking in spreadsheets and prevents deployment failures caused by port conflicts.


# 1 · Deploy Portracker
# {.tabset}
## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Add two **Additional Environment Variables** (see [section 2](https://wiki.serversatho.me/en/portracker#h-2-integrating-with-truenas) for explanation)
	a. TRUENAS_API_KEY
  b. TRUENAS_WS_BASE
1. Set the **Data Storage** to *Host Path*

## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  portracker:
    image: mostafawahied/portracker:latest
    container_name: portracker
    restart: unless-stopped
    pid: "host"
    ports:
    	- 4999:4999
    cap_add:
      - SYS_PTRACE
      - SYS_ADMIN
    security_opt:
      - apparmor:unconfined
    volumes:
      - /mnt/tank/configs/portracker:/data
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      - DATABASE_PATH=/data/portracker.db
      - PORT=4999
      # Optional: For enhanced TrueNAS features
      # - TRUENAS_API_KEY=your-api-key-here
      # - TRUENAS_WS_BASE=wss://10.99.0.191:444
```

# 2 · Integrating With TrueNAS

To generate the `TRUENAS_API_KEY`, click the username in the top right of the web UI and select **My API Keys**. Click **Add** in the top right and generate a new API key, then copy it into the compose file above.

The `TRUENAS_WS_BASE` environment variable needs to be set to the IP of your TrueNAS server and the https port you have set for the webGUI in the format `wss://IP:port`. 

# <img src="/youtube.png" class="tab-icon"> 3 · Video

https://youtu.be/oTG5gBA6UgM