---
title: Pulsarr
description: A guide to deploying Pulsarr via docker
published: true
date: 2025-07-06T10:45:01.625Z
tags: 
editor: markdown
dateCreated: 2025-07-06T09:47:13.858Z
---

![pulsarr.png](/pulsarr.png)

# What is Pulsarr?

Pulsarr is an integration tool that bridges Plex watchlists with Sonarr and Radarr, enabling real-time media monitoring and automated content acquisition all from within the Plex App itself.

# Installation
# {.tabset}
## Docker Compose
```yaml
services:
  pulsarr:
    image: lakker/pulsarr:latest
    container_name: pulsarr
    ports:
      - 3003:3003
    volumes:
      - /mnt/tank/configs/[ulsarr:/app/data
      - .env:/app/.env
    restart: unless-stopped
    env_file:
      - .env
```

### env file
```yaml
baseUrl=http://10.99.0.191 
port=3003                       
TZ=America/New_York         
logLevel=info                  
NODE_ARGS=--log-both  
```

## Docker + Apprise
```yaml
services:
  apprise:
    image: caronc/apprise:latest
    container_name: apprise
    ports:
      - "8000:8000"
    environment:
      - PUID=${PUID:-1000}
      - PGID=${PGID:-1000}
      - APPRISE_STATEFUL_MODE=simple
      - APPRISE_WORKER_COUNT=1
    volumes:
      - /mnt/tank/configs/apprise/config:/config
      - /mnt/tank/configs/apprise/plugin:/plugin
      - /mnt/tank/configs/apprise/attach:/attach
    restart: unless-stopped

  pulsarr:
    image: lakker/pulsarr:latest
    container_name: pulsarr
    ports:
      - "3003:3003"
    volumes:
      - /mnt/tank/configs/[ulsarr:/app/data
      - .env:/app/.env
    restart: unless-stopped
    env_file:
      - .env
    depends_on:
      - apprise
```
### env file
```yaml
baseUrl=http://10.99.0.191 
port=3003                       
TZ=America/New_York         
logLevel=info                  
NODE_ARGS=--log-both
appriseUrl=http://host-ip-address:8000
```
- Replace `host-ip-address` with your actual server IP 

# Pulsarr Configuration
1. Access the web interface at `http://your-server:3003`
1. Create an admin account
1. Enter your Plex token. If you do not know it, login to the webpage (not the container) for Plex then open another browser window and visit https://plex.tv/devices.xml.
1. Configure your Sonarr and Radarr connections:
	a. Add instance details (URL, API key)
	b. Configure default quality profiles and root folders
1. Set sync permissions for any friends' watchlists you'd like to include (Ensure users have their [Account Visibility](https://app.plex.tv/desktop/#!/settings/account) set to 'Friends Only' or 'Friends of Friends')
1. Head to the **Dashboard** page and click on the Start button next to the Main Workflow heading
