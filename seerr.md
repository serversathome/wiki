---
title: Seerr
description: A guide to deploying Seerr
published: true
date: 2026-02-14T20:14:06.416Z
tags: 
editor: markdown
dateCreated: 2026-02-14T20:08:17.652Z
---

# <img src="/seerr.png" class="tab-icon"> What is Seerr?

**Seerr** is the unified successor to both Overseerr and Jellyseerr — a free and open-source media request and discovery manager. It integrates with **Jellyfin**, **Plex**, and **Emby**, along with existing services like **Sonarr** and **Radarr**, giving your users a clean interface to request movies and TV shows.

Seerr v3.0.0 merges the Overseerr and Jellyseerr codebases into a single project, combining all existing Overseerr functionality with the latest Jellyseerr features.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Seerr

```yaml
services:
  seerr:
    image: ghcr.io/seerr-team/seerr:latest
    init: true
    container_name: seerr
    environment:
      - LOG_LEVEL=debug
      - TZ=America/New_York
      - PORT=5055
    ports:
      - 5055:5055
    volumes:
      - /mnt/tank/configs/seerr:/app/config
    healthcheck:
      test: wget --no-verbose --tries=1 --spider http://localhost:5055/api/v1/status || exit 1
      start_period: 20s
      timeout: 3s
      interval: 15s
      retries: 3
    restart: unless-stopped
```

>
> The `init: true` directive is **required**. Seerr no longer provides its own init process inside the container.
{.is-info}

# 2 · Permissions

The Seerr container always runs as UID **1000** (the `node` user) regardless of any `user:` directive in your compose file. On TrueNAS, where the standard `apps` user is UID **568**, this means the config directory must be owned by UID 1000 for Seerr to function properly.

```bash
chown -R 1000:1000 /mnt/tank/configs/seerr
```

> 
> If Seerr is having trouble writing to its config directory, permissions are almost always the cause. Check with `ls -la /mnt/tank/configs/seerr` and verify UID 1000 owns the files.
{.is-info}

# 3 · Migrating from Jellyseerr

If you're coming from Jellyseerr, the migration to Seerr is mostly automatic — but there are a few important differences to be aware of on TrueNAS.


## 3.1 Migration Steps

> 
> Do **not** point Seerr at your existing Jellyseerr dataset. Create a new dataset and copy your data into it. This keeps your Jellyseerr install intact as a rollback option.
{.is-danger}

**1. Stop the Jellyseerr container**

Stop the Jellyseerr stack in Dockge or run:

```bash
docker stop jellyseerr
```

**2. Create a new dataset for Seerr**

In the TrueNAS UI, create a new dataset for Seerr. 

**3. Copy your Jellyseerr config into the new dataset**

Use `rsync` to copy your existing Jellyseerr data into the new Seerr dataset:

```bash
rsync -avhz /mnt/tank/configs/jellyseerr/ /mnt/tank/configs/seerr/
```

> 
> Note the trailing slash on the source path — this copies the **contents** of the directory, not the directory itself.
{.is-warning}

**4. Set ownership to UID 1000**

Jellyseerr ran as root, so the copied files will be owned by `root:root`. Seerr runs as UID 1000 internally and needs ownership of these files:

```bash
chown -R 1000:1000 /mnt/tank/configs/seerr
```

**5. Deploy the Seerr stack**

Create a new stack in Dockge using the compose from [Section 1](https://wiki.serversatho.me/en/seerr#h-1-deploy-seerr), with the volume pointed at your new dataset:

```yaml
    volumes:
      - /mnt/tank/configs/seerr:/app/config
```

Seerr will automatically detect and migrate your Jellyseerr database on first boot.

**6. Verify the migration**

Navigate to `http://your-server-ip:5055` and confirm your settings, users, and request history carried over. Once you're satisfied everything is working, you can remove the old Jellyseerr container and dataset at your discretion.

# 4 · Configuration

## 4.1 PostgreSQL (Optional)

Seerr v3.0.0 adds optional PostgreSQL support. By default, Seerr uses SQLite (stored in the config directory). If you want to switch to PostgreSQL, refer to the [official database configuration docs](https://docs.seerr.dev/extending-jellyseerr/database-config#postgresql-options).

## 4.2 DNS Caching (Experimental)

If you run Pi-hole or AdGuard, Seerr's new DNS caching feature can help reduce DNS query spam from the container. This is experimental and can be enabled in the Seerr settings.

# <img src="/youtube.png" class="tab-icon"> 5 · Video

*Coming soon*