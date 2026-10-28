---
title: OpenCloud
description: A guide to deploying OpenCloud
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-03-22T11:56:59.012Z
---

# <img src="/opencloud.png" class="tab-icon"> What is OpenCloud?

**OpenCloud** is an open-source, self-hosted file sharing and collaboration platform. Built in Go with a Vue.js web frontend, it provides a clean and modern alternative to Nextcloud with features like file syncing, public link sharing, workspaces for team collaboration, file versioning, full-text search, and WebDAV access. OpenCloud supports desktop clients (Windows, macOS, Linux), mobile apps (Android, iOS), and can be extended with Collabora Online for document editing and Radicale for calendars and contacts.

> 
> OpenCloud was created by members of the former ownCloud team and is developed by the Heinlein Gruppe in Germany.
{.is-info}

<div class="glance">
  <div><span>Port</span><b><code>9200</code></b></div>
  <div><span>Deploy via</span><b>TrueNAS app or Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://youtu.be/CfGRm6QtEjE"><i class="mdi mdi-youtube"></i>Video walkthrough</a>
  </div>
</div>

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
      - IDM_CREATE_DEMO_USERS=false
      - IDM_ADMIN_PASSWORD=<strong password> # used by init on first start only
      - PROXY_HTTP_ADDR=0.0.0.0:9200
    restart: unless-stopped
```

1. Replace `cloud.yourdomain.com` with your actual domain
2. Set `IDM_ADMIN_PASSWORD` to a strong password for the `admin` user
3. Set `IDM_CREATE_DEMO_USERS` to `true` if you want demo users for testing

> 
> On first start, `opencloud init` generates the config files and uses `IDM_ADMIN_PASSWORD` to set the admin credentials. If you leave it out, a random password is generated and printed once in the container logs (`docker logs opencloud`). The `|| true` ensures the container doesn't fail if already initialized on subsequent starts.
{.is-info}


### 1.1 Change the Admin Password

Log in as `admin` with the `IDM_ADMIN_PASSWORD` you set. You can change it later by navigating to your profile settings in the OpenCloud web UI.

> 
> `IDM_ADMIN_PASSWORD` is only used on the very first start. After initialization, changing the value in the compose file has no effect — the password can only be changed through the OpenCloud web UI or CLI.
{.is-warning}


## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Navigate to **Apps** in the TrueNAS UI
2. Search for **OpenCloud**
3. Click **Install**
4. Configure the following settings:
   - **Application URL**: `https://cloud.yourdomain.com`
   - **Config Storage** → **Host Path**: `/mnt/tank/configs/opencloud/config`
   - **Data Storage** → **Host Path**: `/mnt/tank/configs/opencloud/data`
   - **Radicale Data Storage** → **Host Path**: `/mnt/tank/configs/opencloud/radicale`
   - Optional: **Full Text Search** (Tika), **Yjs (Real-time Collaboration)** and **Collaboration** (Collabora)
5. Click **Save**
6. Log in as `admin` / `admin` and **change the password immediately**. The TrueNAS app has no admin password field and always initializes the admin account with `admin`

> 
> The TrueNAS Community app includes Radicale (CalDAV/CardDAV) and Tika (full-text search) containers bundled in. You'll still need to configure your Cloudflare Tunnel to point to the exposed port.
{.is-info}



# <img src="/youtube.png" class="tab-icon"> 2 · Video
https://youtu.be/CfGRm6QtEjE
