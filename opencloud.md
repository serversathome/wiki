---
title: OpenCloud
description: A guide to deploying OpenCloud
published: true
date: 2026-04-06T21:20:55.278Z
tags: 
editor: markdown
dateCreated: 2026-03-22T11:56:59.012Z
---

# <img src="/opencloud.png" class="tab-icon"> What is OpenCloud?

**OpenCloud** is an open-source, self-hosted file sharing and collaboration platform. Built in Go with a Vue.js web frontend, it provides a clean and modern alternative to Nextcloud with features like file syncing, public link sharing, workspaces for team collaboration, file versioning, full-text search, and WebDAV access. OpenCloud supports desktop clients (Windows, macOS, Linux), mobile apps (Android, iOS), and can be extended with Collabora Online for document editing and Radicale for calendars and contacts.

> 
> OpenCloud was created by members of the former ownCloud team and is developed by the Heinlein Gruppe in Germany.
{.is-info}

# 1 · Deploy OpenCloud
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

```yaml
services:
  opencloud:
    image: opencloudeu/opencloud-rolling
    container_name: opencloud
    user: "568:568"
    volumes:
      - /mnt/tank/configs/opencloud/data:/var/lib/opencloud
      - /mnt/tank/configs/opencloud/config:/etc/opencloud
    ports:
      - 9200:9200
    entrypoint:
      - /bin/sh
    command: ["-c", "opencloud init --insecure true || true; opencloud server"]
    environment:
      - OC_URL=https://cloud.yourdomain.com
      - INITIAL_ADMIN_PASSWORD=admin
      - IDM_CREATE_DEMO_USERS=false
    restart: unless-stopped
```

1. Replace `cloud.yourdomain.com` with your actual domain
2. Set `IDM_CREATE_DEMO_USERS` to `true` if you want demo users for testing

> 
> On first start, `opencloud init` generates the config files and uses `INITIAL_ADMIN_PASSWORD` to set the admin credentials. The `|| true` ensures the container doesn't fail if already initialized on subsequent starts.
{.is-info}


### 1.1 Change the Admin Password

The default admin password is set to `admin` in the compose file. **Change this immediately** after your first login by navigating to your profile settings in the OpenCloud web UI.

> 
> `INITIAL_ADMIN_PASSWORD` is only used on the very first start. After initialization, changing the value in the compose file has no effect — the password can only be changed through the OpenCloud web UI or CLI.
{.is-warning}


## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Navigate to **Apps** in the TrueNAS UI
2. Search for **OpenCloud**
3. Click **Install**
4. Configure the following settings:
   - **Admin Password**: Set a strong initial admin password
   - **OpenCloud URL**: `https://cloud.yourdomain.com`
   - **Host Path (Config)**: `/mnt/tank/configs/opencloud/config`
   - **Host Path (Data)**: `/mnt/tank/configs/opencloud/data`
   - **Host Path (Radicale Data)**: `/mnt/tank/configs/opencloud/radicale`
5. Click **Save**

> 
> The TrueNAS Community app (v4.0.3) includes Radicale (CalDAV/CardDAV) and Tika (full-text search) containers bundled in. You'll still need to configure your Cloudflare Tunnel to point to the exposed port.
{.is-info}



# <img src="/youtube.png" class="tab-icon"> 2 · Video

*Video coming soon! Check back later.*