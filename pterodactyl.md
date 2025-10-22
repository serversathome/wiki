---
title: Pterodactyl & Wings
description: A guide to deploying Pterodactyl Panel and Wings
published: true
date: 2025-10-22T19:05:55.404Z
tags: 
editor: markdown
dateCreated: 2025-10-22T18:47:24.209Z
---

# ![](/pterodactyl.png){class="tab-icon"} What is Pterodactyl?

Pterodactyl® is a free, open-source game server management panel built with PHP, React, and Go. Designed with security in mind, Pterodactyl runs all game servers in isolated Docker containers while exposing a beautiful and intuitive UI to end users. 

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Pterodactyl
```yaml
services:
  db:
    image: "mariadb:latest"
    container_name: pterodactyl_mariadb
    restart: unless-stopped
    command: "--default-authentication-plugin=mysql_native_password"
    volumes:
      - "${CONFIG_VOLUME}/panel/db:/var/lib/mysql"
    environment:
      MYSQL_DATABASE: panel
      MYSQL_USER: pterodactyl 
      MYSQL_PASSWORD: pterodactyl! 
      MYSQL_ROOT_PASSWORD: pterodactyl!!
    networks:
      - pterodactyl
  
  cache:
    image: "redis:alpine"
    container_name: pterodactyl_redis
    restart: unless-stopped
    networks:
      - pterodactyl

  panel:
    image: "ghcr.io/pterodactyl/panel:latest"
    container_name: pterodactyl_panel
    restart: unless-stopped
    stdin_open: true
    tty: true
    ports:
      - "${PANEL_PORT}:80"
    volumes:
      - "${CONFIG_VOLUME}/panel/var/:/app/var/"
      - "${CONFIG_VOLUME}/panel/logs/:/app/storage/logs"
      - "${CONFIG_VOLUME}/panel/nginx/:/etc/nginx/conf.d/"
    environment:
      RECAPTCHA_ENABLED: false
      TZ: ${TZ}
      APP_TIMEZONE: ${TZ}
      APP_ENV: production
      APP_ENVIRONMENT_ONLY: false
      APP_URL: "${FQDN}"
      APP_SERVICE_AUTHOR: example@gmail.com # Replace with your email
      TRUSTED_PROXIES: "*"
      PTERODACTYL_TELEMETRY_ENABLED: false
      DB_HOST: db
      DB_PORT: 3306
      DB_USERNAME: pterodactyl 
      DB_PASSWORD: pterodactyl
      CACHE_DRIVER: redis
      SESSION_DRIVER: redis
      QUEUE_DRIVER: redis
      REDIS_HOST: cache
    networks:
      - pterodactyl

  wings:
    image: "ghcr.io/pterodactyl/wings:latest"
    container_name: pterodactyl_wings
    restart: unless-stopped
    ports:
      - "2022:2022"
      - "8443:443"
    stdin_open: true
    tty: true
    environment:
      TZ: ${TZ}
      APP_TIMEZONE: ${TZ}
      WINGS_UID: 1000
      WINGS_GID: 1000
      WINGS_USERNAME: pterodactyl
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
      - "${CONFIG_VOLUME}/var/lib/docker/containers/:/var/lib/docker/containers/"
      - "${CONFIG_VOLUME}/etc/pterodactyl/:/etc/pterodactyl/"
      - "${CONFIG_VOLUME}/var/lib/pterodactyl/:/var/lib/pterodactyl/"
      - "${CONFIG_VOLUME}/var/log/pterodactyl/:/var/log/pterodactyl/"
      - "${CONFIG_VOLUME}/tmp/pterodactyl/:/tmp/pterodactyl/"
      - "${CONFIG_VOLUME}/etc/ssl/certs:/etc/ssl/certs:ro"
    networks:
      - wings0

networks:
  pterodactyl:
    name: pterodactyl
  wings0:
    name: wings0
    driver: bridge
    ipam:
      config:
        - subnet: 172.50.0.0/16
    driver_opts:
      com.docker.network.bridge.name: wings0
```

# 1.1 env Variables

```yaml
CONFIG_VOLUME=/mnt/tank/configs/pterodactyl
PANEL_PORT=8080
TZ=America/New_York
```