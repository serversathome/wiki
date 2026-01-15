---
title: Pulsarr
description: A guide to deploying Pulsarr via docker
published: true
date: 2026-01-15T15:31:04.804Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:37.846Z
---

# ![](/pulsarr.png){class="tab-icon"} What is Pulsarr?

Pulsarr is an integration tool that bridges Plex watchlists with Sonarr and Radarr, enabling real-time media monitoring and automated content acquisition all from within the Plex App itself.

# 1 · Deploy Pulsarr
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose
```yaml
services:
  pulsarr:
    image: lakker/pulsarr:latest
    container_name: pulsarr
    ports:
      - 3003:3003
    volumes:
      - /mnt/tank/configs/pulsarr:/app/data
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
- Replace the `baseUrl` with your IP address

## <img src="/docker.png" class="tab-icon"> Docker Compose + Apprise
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
      - /mnt/tank/configs/pulsarr:/app/data
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
- Replace the `baseUrl` with your IP address
- Replace `host-ip-address` with your actual server IP 

# 2 · Pulsarr Configuration
> Read the [official documentation](https://jamcalli.github.io/Pulsarr/docs/intro)
{.is-success}


1. Access the web interface at `http://your-server:3003`
1. Create an admin account
1. Enter your [Plex token](https://support.plex.tv/articles/204059436-finding-an-authentication-token-x-plex-token/)
1. Configure your Sonarr and Radarr connections:
	a. Add instance details (URL, API key)
	b. Configure default quality profiles and root folders
1. Set sync permissions for any friends' watchlists you'd like to include (Ensure users have their [Account Visibility](https://app.plex.tv/desktop/#!/settings/account) set to 'Friends Only' or 'Friends of Friends')
1. Head to the **Dashboard** page and click on the Start button next to the Main Workflow heading

# <img src="/patreon-light.png" class="tab-icon"> 3 · Video
[![](/2025-07-06-automate-your-plex-watchlist-wit-promo-card.png)](https://www.patreon.com/posts/automate-your-133499239)