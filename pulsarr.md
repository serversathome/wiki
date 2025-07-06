---
title: Pulsarr
description: A guide to deploying Pulsarr via docker
published: true
date: 2025-07-06T09:47:13.858Z
tags: 
editor: markdown
dateCreated: 2025-07-06T09:47:13.858Z
---

![pulsarr.png](/pulsarr.png)

# What is Pulsarr?

Pulsarr is an integration tool that bridges Plex watchlists with Sonarr and Radarr, enabling real-time media monitoring and automated content acquisition all from within the Plex App itself.

# Installation
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

## env file
```yaml
baseUrl=http://10.99.0.191 
port=3003                       
TZ=America/New_York         
logLevel=info                  
NODE_ARGS=--log-both  
```

# Pulsarr Configuration
