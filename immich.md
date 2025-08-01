---
title: Immich
description: A guide to deploying Immich on TrueNAS and via docker
published: true
date: 2025-08-01T20:56:16.752Z
tags: 
editor: markdown
dateCreated: 2025-04-30T12:06:15.920Z
---

# ![](/immich.png){class="tab-icon"} What is Immich?
Easily back up, organize, and manage your photos on your own server. Immich helps you browse, search and organize your photos and videos with ease, without sacrificing your privacy.

# 1 · Deploy Immich
# {.tabset}


## <img src="/truenas.png" class="tab-icon"> TrueNAS
![screenshot_from_2025-04-30_08-01-41.png](/screenshot_from_2025-04-30_08-01-41.png)

1. Set a database and redis password
1. If you have an nVidia GPU, select the **Cuda Machine Learning Image** for the **Machine Learning Image Type**
1. Set **Host Path Storage** for **Data** and **Postgres Storage** options

> Since Immich runs as `root` **Generic permissions** are fine for all of these datasets
{.is-info}

> The **Postgres Data Storage** needs the **☑ Automatic Permissions** checkbox checked!
{.is-warning}

4. If you can, increase limits on the **Resources Configuration**

> Check out [the new docs](https://apps.truenas.com/resources/deploy-immich) from TrueNAS
{.is-success}

## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    volumes:
      - /mnt/tank/configs/immich/uploads:/data
      - /etc/localtime:/etc/localtime:ro
    ports:
      - '2283:2283'
    depends_on:
      - redis
      - database
    restart: unless-stopped
    healthcheck:
      disable: false

  immich-machine-learning:
    container_name: immich_machine_learning
    image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}
    volumes:
      - ./model-cache:/cache
    restart: unless-stopped
    healthcheck:
      disable: false

  redis:
    container_name: immich_redis
    image: docker.io/valkey/valkey:8-bookworm@sha256:42cba146593a5ea9a622002c1b7cba5da7be248650cbb64ecb9c6c33d29794b1
    healthcheck:
      test: redis-cli ping || exit 1
    restart: unless-stopped

  database:
    container_name: immich_postgres
    image: docker.io/tensorchord/pgvecto-rs:pg14-v0.2.0@sha256:739cdd626151ff1f796dc95a6591b55a714f341c737e27f045019ceabf8e8c52
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
      POSTGRES_INITDB_ARGS: '--data-checksums'
    volumes:
      - /mnt/tank/configs/immich/db:/var/lib/postgresql/data
    healthcheck:
      test: >-
        pg_isready --dbname="$${POSTGRES_DB}" --username="$${POSTGRES_USER}" || exit 1; Chksum="$$(psql --dbname="$${POSTGRES_DB}" --username="$${POSTGRES_USER}" --tuples-only --no-align --command='SELECT COALESCE(SUM(checksum_failures), 0) FROM pg_stat_database')"; echo "checksum failure count is $$Chksum"; [ "$$Chksum" = '0' ] || exit 1
      interval: 5m
      start_interval: 30s
      start_period: 5m
    command: >-
      postgres -c shared_preload_libraries=vectors.so -c 'search_path="$$user", public, vectors' -c logging_collector=on -c max_wal_size=2GB -c shared_buffers=512MB -c wal_compression=on
    restart: unless-stopped
```
 ### env Variables
 ```yaml
IMMICH_VERSION=release
DB_PASSWORD=postgres
DB_USERNAME=postgres
DB_DATABASE_NAME=immich
 ```
 Set the `IMMICH_VERSION` based on the releases [here](https://github.com/immich-app/immich/releases/).
 
### Folder Structure
```xml
tank
 └── configs
   └── immich
        ├── uploads
        └── db
```
- The `uploads` directory will be where all photos are stored in individual folders per user

> Since Immich runs as `root` **Generic permissions** are fine for all of these datasets
{.is-info}

## <img src="/docker.png" class="tab-icon"> Docker Compose with nVidia GPU

```yaml
services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    volumes:
      - /mnt/tank/configs/immich/uploads:/data
      - /etc/localtime:/etc/localtime:ro
    ports:
      - 2283:2283
    depends_on:
      - redis
      - database
    restart: unless-stopped
    healthcheck:
      disable: false
  immich-machine-learning:
    container_name: immich_machine_learning
    image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}-cuda
    volumes:
      - /mnt/tank/configs/immich/ml:/cache
    restart: unless-stopped
    healthcheck:
      disable: false
    environment:
      - MACHINE_LEARNING__FACE_RECOGNITION_MODEL=mobile_face
      - MACHINE_LEARNING__FACE_DETECTION_MODEL=yunet
      - MACHINE_LEARNING__CLIP_MODEL=ViT-B-16__laion2b_s34b_b88k
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities:
                - gpu
  redis:
    container_name: immich_redis
    image: docker.io/valkey/valkey:8-bookworm@sha256:42cba146593a5ea9a622002c1b7cba5da7be248650cbb64ecb9c6c33d29794b1
    healthcheck:
      test: redis-cli ping || exit 1
    restart: unless-stopped
  database:
    container_name: immich_postgres
    image: docker.io/tensorchord/pgvecto-rs:pg14-v0.2.0@sha256:739cdd626151ff1f796dc95a6591b55a714f341c737e27f045019ceabf8e8c52
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
      POSTGRES_INITDB_ARGS: --data-checksums
    volumes:
      - /mnt/tank/configs/immich/db:/var/lib/postgresql/data
    healthcheck:
      test: pg_isready --dbname="$${POSTGRES_DB}" --username="$${POSTGRES_USER}" ||
        exit 1; Chksum="$$(psql --dbname="$${POSTGRES_DB}"
        --username="$${POSTGRES_USER}" --tuples-only --no-align
        --command='SELECT COALESCE(SUM(checksum_failures), 0) FROM
        pg_stat_database')"; echo "checksum failure count is $$Chksum"; [
        "$$Chksum" = '0' ] || exit 1
      interval: 5m
      start_interval: 30s
      start_period: 5m
    command: postgres -c shared_preload_libraries=vectors.so -c
      'search_path="$$user", public, vectors' -c logging_collector=on -c
      max_wal_size=2GB -c shared_buffers=512MB -c wal_compression=on
    restart: unless-stopped
```
 ### env Variables
 ```yaml
IMMICH_VERSION=release
DB_PASSWORD=postgres
DB_USERNAME=postgres
DB_DATABASE_NAME=immich
 ```
 Set the `IMMICH_VERSION` based on the releases [here](https://github.com/immich-app/immich/releases/).
 
### Folder Structure
```xml
tank
 └── configs
   └── immich
        ├── uploads
        ├── ml
        └── db
```
# 2 · Adding External Libraries
1. If installing from the TrueNAS apps catalog, under **Storage Configuration →  Additional Storage**, add a host path pointed to where your photos are stored
1. If installing via docker compose, add an additional line in the `immich_server` section pointed to where your photos are stored
1. Click the circle with the letter in the top right and select **Administration**
1. Select **External Libraries** in the left panel
1. Select an external library owner
1. Click the 3 dots at the end of the row which was just created
1. Select **Edit Import Paths**
1. Select **Add Path**
1. Add the path you mounted earlier and click **Add**
1. Click **Save**

# 3 · Migrating to New Dataset Structure

1. Create new dataset (I am calling them `immich1` for simplicity) with **apps** permission preset
1. Create two sub-datasets: `data` and `db` with **apps** permission preset
1. Use `rsync` to copy data from the old datasets to the new ones (assuming this is how your old dataset structure is named)
    ```bash
    rsync -avhz --progress /mnt/tank/configs/immich/backups/ /mnt/tank/configs/immich1/data/backups/ && \
    rsync -avhz --progress /mnt/tank/configs/immich/library/ /mnt/tank/configs/immich1/data/library/ && \
    rsync -avhz --progress /mnt/tank/configs/immich/profile/ /mnt/tank/configs/immich1/data/profile/ && \
    rsync -avhz --progress /mnt/tank/configs/immich/thumbs/ /mnt/tank/configs/immich1/data/thumbs/ && \
    rsync -avhz --progress /mnt/tank/configs/immich/video/ /mnt/tank/configs/immich1/data/encoded-video/ && \
    rsync -avhz --progress /mnt/tank/configs/immich/uploads/ /mnt/tank/configs/immich1/data/upload/ && \
    rsync -avhz --progress /mnt/tank/configs/immich/db/ /mnt/tank/configs/immich1/db/
    ```
1. Edit Immich app
a. Scroll down to **Storage Configuration** and uncheck the box **Use Old Storage Configuration (Deprecated)**
b. Set the **Data Storage** to the new dataset `/mnt/tank/immich1/data`
c. Set the **Postgres Data Storage** to the new dataset `/mnt/tank/immich1/db` and be sure to check the **Automatic Permissions** checkbox
d. Click the blue **Update** button

# <img src="/youtube.png" class="tab-icon"> 4 · Video
[](https://youtu.be/TqjlUocu6ZI)
