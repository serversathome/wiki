---
title: Immich
description: A guide to deploying Immich on TrueNAS and via docker
published: true
date: 2025-08-03T19:12:13.282Z
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

## <img src="/docker.png" class="tab-icon"> Docker Compose - CPU

```yaml
services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    volumes:
      - ${UPLOAD_LOCATION}:/data
      - /etc/localtime:/etc/localtime:ro
    env_file:
      - .env
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
      - model-cache:/cache
    env_file:
      - .env
    restart: unless-stopped
    healthcheck:
      disable: false

  redis:
    container_name: immich_redis
    image: docker.io/valkey/valkey:8-bookworm@sha256:fea8b3e67b15729d4bb70589eb03367bab9ad1ee89c876f54327fc7c6e618571
    healthcheck:
      test: redis-cli ping || exit 1
    restart: unless-stopped

  database:
    container_name: immich_postgres
    image: ghcr.io/immich-app/postgres:14-vectorchord0.4.3-pgvectors0.2.0@sha256:bcf63357191b76a916ae5eb93464d65c07511da41e3bf7a8416db519b40b1c23
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
      POSTGRES_INITDB_ARGS: '--data-checksums'
      # Uncomment the DB_STORAGE_TYPE: 'HDD' var if your database isn't stored on SSDs
      # DB_STORAGE_TYPE: 'HDD'
    volumes:
      - ${DB_DATA_LOCATION}:/var/lib/postgresql/data
    shm_size: 128mb
    restart: unless-stopped

volumes:
  model-cache:
```

### env Variables

 ```yaml
UPLOAD_LOCATION=/mnt/tank/configs/immich/uploads
IMMICH_VERSION=release
DB_DATA_LOCATION=/mnt/tank/configs/immich/db
DB_PASSWORD=postgres
DB_USERNAME=postgres
DB_DATABASE_NAME=immich
 ```

> Make sure to change the `DB_PASSWORD` to something random!
{.is-danger}

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

## <img src="/docker.png" class="tab-icon"> Docker Compose - Nvidia GPU

```yaml
services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities:
                - gpu
                - compute
                - video
    volumes:
      - ${UPLOAD_LOCATION}:/data
      - /etc/localtime:/etc/localtime:ro
    env_file:
      - .env
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
    image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}-cuda
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities:
                - gpu
    volumes:
      - model-cache:/cache
    env_file:
      - .env
    restart: unless-stopped
    healthcheck:
      disable: false

  redis:
    container_name: immich_redis
    image: docker.io/valkey/valkey:8-bookworm@sha256:fea8b3e67b15729d4bb70589eb03367bab9ad1ee89c876f54327fc7c6e618571
    healthcheck:
      test: redis-cli ping || exit 1
    restart: unless-stopped

  database:
    container_name: immich_postgres
    image: ghcr.io/immich-app/postgres:14-vectorchord0.4.3-pgvectors0.2.0@sha256:bcf63357191b76a916ae5eb93464d65c07511da41e3bf7a8416db519b40b1c23
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
      POSTGRES_INITDB_ARGS: '--data-checksums'
      # Uncomment the DB_STORAGE_TYPE: 'HDD' var if your database isn't stored on SSDs
      # DB_STORAGE_TYPE: 'HDD'
    volumes:
      - ${DB_DATA_LOCATION}:/var/lib/postgresql/data
    shm_size: 128mb
    restart: unless-stopped

volumes:
  model-cache:
```

### env Variables

 ```yaml
UPLOAD_LOCATION=/mnt/tank/configs/immich/uploads
IMMICH_VERSION=release
DB_DATA_LOCATION=/mnt/tank/configs/immich/db
DB_PASSWORD=postgres
DB_USERNAME=postgres
DB_DATABASE_NAME=immich
MACHINE_LEARNING_MODEL_TTL=0
MACHINE_LEARNING_MODEL_TTL_POLL_S=0
 ```

> Make sure to change the `DB_PASSWORD` to something random!
{.is-danger}

Set the `IMMICH_VERSION` based on the releases [here](https://github.com/immich-app/immich/releases/).

> In case you want to use multiple gpus see: [https://docs.immich.app/features/ml-hardware-acceleration#multi-gpu](https://docs.immich.app/features/ml-hardware-acceleration#multi-gpu)
> The GPU must have compute capability 5.2 or greater.
> The server must have the official NVIDIA driver installed.
> The installed driver must be >= 545 (it must support CUDA 12.3).
{.is-info}

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

### Confirming GPU Utilization

To confirm you are actually using your GPU, you should do the following:

1. On TrueNAS go to System > Shell
2. Run `watch -d -n 1 nvidia-smi` inside the terminal
3. You should be able to see a python process running if there is a job running for Smart Search, Facial Detection or Facial Recognition

This is under assumption that you have some content in your library.

## <img src="/docker.png" class="tab-icon"> Docker Compose - AMD GPU

```yaml
services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    devices:
      - /dev/dri:/dev/dri
    volumes:
      - ${UPLOAD_LOCATION}:/data
      - /etc/localtime:/etc/localtime:ro
    env_file:
      - .env
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
    image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}-rocm
    group_add:
      - video
    devices:
      - /dev/dri:/dev/dri
      - /dev/kfd:/dev/kfd
    volumes:
      - model-cache:/cache
    env_file:
      - .env
    restart: unless-stopped
    healthcheck:
      disable: false

  redis:
    container_name: immich_redis
    image: docker.io/valkey/valkey:8-bookworm@sha256:fea8b3e67b15729d4bb70589eb03367bab9ad1ee89c876f54327fc7c6e618571
    healthcheck:
      test: redis-cli ping || exit 1
    restart: unless-stopped

  database:
    container_name: immich_postgres
    image: ghcr.io/immich-app/postgres:14-vectorchord0.4.3-pgvectors0.2.0@sha256:bcf63357191b76a916ae5eb93464d65c07511da41e3bf7a8416db519b40b1c23
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
      POSTGRES_INITDB_ARGS: '--data-checksums'
      # Uncomment the DB_STORAGE_TYPE: 'HDD' var if your database isn't stored on SSDs
      # DB_STORAGE_TYPE: 'HDD'
    volumes:
      - ${DB_DATA_LOCATION}:/var/lib/postgresql/data
    shm_size: 128mb
    restart: unless-stopped

volumes:
  model-cache:
```

> The GPU must be supported by ROCm. If it isn't officially supported, you can attempt to use the `HSA_OVERRIDE_GFX_VERSION` environmental variable: `HSA_OVERRIDE_GFX_VERSION=<a supported version, e.g. 10.3.0>`. If this doesn't work, you might need to also set `HSA_USE_SVM=0`.
> The ROCm image is quite large and requires at least 35GiB of free disk space. However, pulling later updates to the service through Docker will generally only amount to a few hundred megabytes as the rest will be cached.
> This backend is new and may experience some issues. For example, GPU power consumption can be higher than usual after running inference, even if the machine learning service is idle. In this case, it will only go back to normal after being idle for 5 minutes (configurable with the [MACHINE_LEARNING_MODEL_TTL](https://docs.immich.app/install/environment-variables) setting).
{.is-info}

### env Variables

 ```yaml
UPLOAD_LOCATION=/mnt/tank/configs/immich/uploads
IMMICH_VERSION=release
DB_DATA_LOCATION=/mnt/tank/configs/immich/db
DB_PASSWORD=postgres
DB_USERNAME=postgres
DB_DATABASE_NAME=immich
MACHINE_LEARNING_MODEL_TTL=0
MACHINE_LEARNING_MODEL_TTL_POLL_S=0
 ```

> Make sure to change the `DB_PASSWORD` to something random!
{.is-danger}

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

### Confirming GPU Utilization

To confirm you are actually using your GPU, you should do the following:

1. On TrueNAS go to System > Shell
2. Run `docker exec -it immich_machine_learning bash` to enter the container
3. First you will have to update the repositories and install radeontop package by running

```sh
apt update && apt install radeontop
```

4. Run `radeontop` inside the terminal
5. You should be able to see usage increasing if there is a job running for Smart Search, Facial Detection or Facial Recognition

This is under assumption that you have some content in your library.

## <img src="/docker.png" class="tab-icon"> Docker Compose - Intel GPU/iGPU

```yaml
services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    devices:
      - /dev/dri:/dev/dri
    volumes:
      - ${UPLOAD_LOCATION}:/data
      - /etc/localtime:/etc/localtime:ro
    env_file:
      - .env
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
    image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}-openvino
    group_add:
      - video
    device_cgroup_rules:
      - 'c 189:* rmw'
    devices:
      - /dev/dri:/dev/dri
    volumes:
      - model-cache:/cache
      - /dev/bus/usb:/dev/bus/usb
    env_file:
      - .env
    restart: unless-stopped
    healthcheck:
      disable: false

  redis:
    container_name: immich_redis
    image: docker.io/valkey/valkey:8-bookworm@sha256:fea8b3e67b15729d4bb70589eb03367bab9ad1ee89c876f54327fc7c6e618571
    healthcheck:
      test: redis-cli ping || exit 1
    restart: unless-stopped

  database:
    container_name: immich_postgres
    image: ghcr.io/immich-app/postgres:14-vectorchord0.4.3-pgvectors0.2.0@sha256:bcf63357191b76a916ae5eb93464d65c07511da41e3bf7a8416db519b40b1c23
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
      POSTGRES_INITDB_ARGS: '--data-checksums'
      # Uncomment the DB_STORAGE_TYPE: 'HDD' var if your database isn't stored on SSDs
      # DB_STORAGE_TYPE: 'HDD'
    volumes:
      - ${DB_DATA_LOCATION}:/var/lib/postgresql/data
    shm_size: 128mb
    restart: unless-stopped

volumes:
  model-cache:
```

> Integrated GPUs are more likely to experience issues than discrete GPUs, especially for older processors or servers with low RAM.
> Ensure the server's kernel version is new enough to use the device for hardware accceleration.
> Expect higher RAM usage when using OpenVINO compared to CPU processing.
{.is-info}

### env Variables

 ```yaml
UPLOAD_LOCATION=/mnt/tank/configs/immich/uploads
IMMICH_VERSION=release
DB_DATA_LOCATION=/mnt/tank/configs/immich/db
DB_PASSWORD=postgres
DB_USERNAME=postgres
DB_DATABASE_NAME=immich
MACHINE_LEARNING_MODEL_TTL=0
MACHINE_LEARNING_MODEL_TTL_POLL_S=0
 ```

> Make sure to change the `DB_PASSWORD` to something random!
{.is-danger}

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

### Confirming GPU Utilization

To confirm you are actually using your GPU, you should do the following:

1. On TrueNAS go to System > Shell
2. Run `intel_gpu_top` inside the terminal
3. You should be able to see a python process running if there is a job running for Smart Search, Facial Detection or Facial Recognition

This is under assumption that you have some content in your library.

## <img src="/docker.png" class="tab-icon"> Docker Compose - VAAPI (Intel/AMD/Nvidia)

```yaml
services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    devices:
      - /dev/dri:/dev/dri
    volumes:
      - ${UPLOAD_LOCATION}:/data
      - /etc/localtime:/etc/localtime:ro
    env_file:
      - .env
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
      - model-cache:/cache
    env_file:
      - .env
    restart: unless-stopped
    healthcheck:
      disable: false

  redis:
    container_name: immich_redis
    image: docker.io/valkey/valkey:8-bookworm@sha256:fea8b3e67b15729d4bb70589eb03367bab9ad1ee89c876f54327fc7c6e618571
    healthcheck:
      test: redis-cli ping || exit 1
    restart: unless-stopped

  database:
    container_name: immich_postgres
    image: ghcr.io/immich-app/postgres:14-vectorchord0.4.3-pgvectors0.2.0@sha256:bcf63357191b76a916ae5eb93464d65c07511da41e3bf7a8416db519b40b1c23
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
      POSTGRES_INITDB_ARGS: '--data-checksums'
      # Uncomment the DB_STORAGE_TYPE: 'HDD' var if your database isn't stored on SSDs
      # DB_STORAGE_TYPE: 'HDD'
    volumes:
      - ${DB_DATA_LOCATION}:/var/lib/postgresql/data
    shm_size: 128mb
    restart: unless-stopped

volumes:
  model-cache:
```

> In case your graphics card, dedicated or integrated, doesn't support machine learning, you can still accelerate transcoding!
>
> You may want to choose a slower preset than for software transcoding to maintain quality and efficiency
> While you can use VAAPI with NVIDIA and Intel devices, prefer the more specific APIs since they're more optimized for their respective devices
{.is-info}

### env Variables

 ```yaml
UPLOAD_LOCATION=/mnt/tank/configs/immich/uploads
IMMICH_VERSION=release
DB_DATA_LOCATION=/mnt/tank/configs/immich/db
DB_PASSWORD=postgres
DB_USERNAME=postgres
DB_DATABASE_NAME=immich
 ```

> Make sure to change the `DB_PASSWORD` to something random!
{.is-danger}

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
Assuming you have the following legacy datasets:
```xml
tank
 └── configs
   └── immich
        ├── backups [dataset]
        ├── library [dataset]
        ├── profile [dataset]
        ├── thumbs [dataset]
        ├── video [dataset]
        ├── uploads [dataset]
        └── db [dataset]
```
You need to migrate to the new datasets, where `data` has the sub-directories `{backups, library, profile, thumbs, encoded_video, upload}`. **Sub-directory naming is important**, and will break the app if not done correctly. I am calling the new parent dataset `immich1` for clarity.
```xml
tank
 └── configs
   └── immich1
        ├── data [dataset]
             ├── backups [directory]
             ├── library [directory]
             ├── profile [directory]
             ├── thumbs [directory]
             ├── encoded_video [directory]
             └── upload [directory]
        └── db [dataset]
```

1. Create new parent dataset (I am calling this `immich1` for simplicity & clarity) with **apps** permission preset
1. Create two sub-datasets: `data` and `db` with **apps** permission preset
1. Use `rsync` to copy data from the old datasets to the new ones (assuming this is how your old dataset structure is named). `rsync` will create the new sub-directories when it completes the sync task. Run the following command in the TrueNAS shell as `root`:
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
1. Once all running, you can remove the old dataset `immich`

# 4 · Picking the right CLIP model

[Immich's documentation](https://docs.immich.app/features/searching#clip-models) is very thorough regarding this so you should just read from there.
Many users don't even know that they can change the model and that it will impact their users significantly, especially if they are not English speakers.

Main things you want to pay attention to is if you need Multilingual model and model size so you are sure it will fit inside your GPU.
Immich team also provided a list of languages that are supported by the Multilingual models and how well they perform.

* `nllb` models expect the search query to be in the language specified in the user settings
* `xlm` and `siglip2` models understand search text regardless of the current language setting

# <img src="/youtube.png" class="tab-icon"> 4 · Video
[](https://youtu.be/TqjlUocu6ZI)
