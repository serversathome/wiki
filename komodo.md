---
title: Komodo
description: A guide to deploying Komodo
published: true
date: 2025-09-04T15:20:31.718Z
tags: 
editor: markdown
dateCreated: 2025-09-01T21:20:12.233Z
---

# ![](/komodo.png){class="tab-icon"} What is Komodo?
Komodo is a web app to provide structure for managing your servers, builds, deployments, and automated procedures.

With Komodo you can:

- Connect all of your servers, alert on CPU usage, memory usage, and disk usage, and connect to shell sessions.
- Create, start, stop, and restart Docker containers on the connected servers, view their status and logs, and connect to container shell.
- Deploy docker compose stacks. The file can be defined in UI, or in a git repo, with auto deploy on git push.
- Build application source into auto-versioned Docker images, auto built on webhook. Deploy single-use AWS instances for infinite capacity.
- Manage repositories on connected servers, which can perform automation via scripting / webhooks.
- Manage all your configuration / environment variables, with shared global variable and secret interpolation.
- Keep a record of all the actions that are performed and by whom.



# 1 · Deploy Komodo
# {.tabset}
## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Set a **Database Password**
1. Set a **Passkey (Periphery)**
1. Set **Backups Storage**, **Syncs Storage**, **Periphery Root Storage**, and **MongoDB Data Storage** to **Host Path**

> Remember to check the box for **Automatic Permissions** under **MongoDB Data Storage**
{.is-warning}



## <img src="/docker.png" class="tab-icon"> Docker Compose


```yaml
services:
  mongo:
    image: mongo
    command: --quiet --wiredTigerCacheSizeGB 0.25
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/komodo/mongo-data:/data/db
      - /mnt/tank/configs/komodo/mongo-config:/data/configdb
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${KOMODO_DB_USERNAME}
      MONGO_INITDB_ROOT_PASSWORD: ${KOMODO_DB_PASSWORD}
  
  core:
    image: ghcr.io/moghtech/komodo-core:${COMPOSE_KOMODO_IMAGE_TAG:-latest}
    restart: unless-stopped
    depends_on:
      - mongo
    ports:
      - 9120:9120
    env_file: ./.env
    environment:
      KOMODO_DATABASE_ADDRESS: mongo:27017
      KOMODO_DATABASE_USERNAME: ${KOMODO_DB_USERNAME}
      KOMODO_DATABASE_PASSWORD: ${KOMODO_DB_PASSWORD}
    volumes:
      - ${COMPOSE_KOMODO_BACKUPS_PATH}:/backups

  periphery:
    image: ghcr.io/moghtech/komodo-periphery:${COMPOSE_KOMODO_IMAGE_TAG:-latest}
    restart: unless-stopped
    env_file: ./.env
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /proc:/proc
      - ${PERIPHERY_ROOT_DIRECTORY:-/etc/komodo}:${PERIPHERY_ROOT_DIRECTORY:-/etc/komodo}
```

### Environment Variables

> If you are using Dockge, place these directly underneath the `compose.yaml` section where is says `.env`
{.is-info}

```yaml
COMPOSE_KOMODO_IMAGE_TAG=latest

COMPOSE_KOMODO_BACKUPS_PATH=/mnt/tank/configs/komodo/backups

## DB credentials
KOMODO_DB_USERNAME=admin
KOMODO_DB_PASSWORD=admin

## Configure a secure passkey to authenticate between Core / Periphery.
KOMODO_PASSKEY=a_random_passkey

TZ=America/New_York

KOMODO_HOST=https://demo.komo.do

KOMODO_TITLE=Komodo

KOMODO_FIRST_SERVER=https://periphery:8120

KOMODO_FIRST_SERVER_NAME=Local

KOMODO_DISABLE_CONFIRM_DIALOG=false

KOMODO_MONITORING_INTERVAL="15-sec"

KOMODO_RESOURCE_POLL_INTERVAL="1-hr"

KOMODO_WEBHOOK_SECRET=a_random_secret

KOMODO_JWT_SECRET=a_random_jwt_secret

KOMODO_LOCAL_AUTH=true

KOMODO_INIT_ADMIN_USERNAME=admin

KOMODO_INIT_ADMIN_PASSWORD=changeme

KOMODO_DISABLE_USER_REGISTRATION=false

KOMODO_ENABLE_NEW_USERS=false

KOMODO_DISABLE_NON_ADMIN_CREATE=false

KOMODO_TRANSPARENT_MODE=false


KOMODO_LOGGING_PRETTY=true

KOMODO_PRETTY_STARTUP_CONFIG=true


KOMODO_OIDC_ENABLED=false
KOMODO_GITHUB_OAUTH_ENABLED=false
KOMODO_GOOGLE_OAUTH_ENABLED=false


KOMODO_AWS_ACCESS_KEY_ID= # Alt: KOMODO_AWS_ACCESS_KEY_ID_FILE
KOMODO_AWS_SECRET_ACCESS_KEY= # Alt: KOMODO_AWS_SECRET_ACCESS_KEY_FILE




PERIPHERY_ROOT_DIRECTORY=/mnt/tank/configs/etc/komodo

PERIPHERY_PASSKEYS=${KOMODO_PASSKEY}

PERIPHERY_DISABLE_TERMINALS=false

PERIPHERY_SSL_ENABLED=true


PERIPHERY_INCLUDE_DISK_MOUNTS=/etc/hostname


PERIPHERY_LOGGING_PRETTY=true

PERIPHERY_PRETTY_STARTUP_CONFIG=true
```