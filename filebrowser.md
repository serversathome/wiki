---
title: File Browser
description: A guide to deploy the File Browser Quantum replacement in docker
published: true
date: 2026-06-01T12:37:48.864Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:39.442Z
---

# ![](/filebrowser-quantum.png){class="tab-icon"} What is File Browser Quantum?
File Browser Quantum provides a file managing interface within a specified directory and it can be used to upload, delete, preview, rename, edit, search, and share your files from the web. It's a modern, responsive file manager with user management, granular access control, and rich media previews — all packed into a single tiny binary.

# This is not the old File Browser
The original File Browser repository (`filebrowser/filebrowser`) is no longer accepting pull requests and is in [maintenance-only mode](https://github.com/filebrowser/filebrowser/discussions/4906#discussioncomment-13436994).

[gtsteffaniak](https://github.com/gtsteffaniak/filebrowser) forked it and built a much better version called **Quantum**. When I first looked at this it was still beta — that's no longer the case. It now ships proper stable releases (latest is **v1.2.2-stable**) and has grown into a genuinely full-featured file manager.

> Quantum adds a lot over the original: multiple sources at once, OIDC / LDAP / JWT / password + 2FA login, ultra-fast SQLite indexed real-time search, thumbnails for office / video / album art / 3D models, highly configurable sharing, directory-level access control, and long-lived API tokens with a built-in Swagger page. One thing worth knowing: shell commands were intentionally removed.
{.is-info}

📌 Full official documentation now lives at [filebrowserquantum.com](https://filebrowserquantum.com/).

# <img src="/docker.png" class="tab-icon"> 1 · Create the config file
Quantum is driven by a `config.yaml` file. Create it **before** you deploy, inside the directory you'll mount as the data volume — so `/mnt/tank/configs/filebrowser/config.yaml`:
```yaml
server:
  port: 80
  baseURL: "/"
  cacheDir: /home/filebrowser/data/tmp   # keep generated thumbnails on the data volume
  logging:
    - levels: "info|warning|error"
  sources:
    - path: "/srv"                        # maps to /mnt below — your whole pool
      config:
        defaultEnabled: true              # add this source for all users by default
userDefaults:
  darkMode: true
  singleClick: true
  disableSettings: false
  preview:
    image: true
    video: false
    office: false
    popup: true
  permissions:
    admin: true
    modify: true
    share: true
    api: true
```
> Quantum now **strictly validates** the config — any unknown key (a typo, or an option that was removed in a newer release) will stop the container from starting rather than being silently ignored. If a deploy fails, check the logs for the offending field name.
{.is-warning}

There are many more options available. See the [full config reference](https://filebrowserquantum.com/en/docs/reference/fullconfig/) or the [annotated default config](https://github.com/gtsteffaniak/filebrowser/blob/main/frontend/public/config.generated.yaml) for everything you can set.

# 2 · Deploy File Browser
```yaml
services:
  filebrowser:
    image: gtstef/filebrowser:latest
    container_name: filebrowser
    volumes:
      - /mnt:/srv
      - /mnt/tank/configs/filebrowser:/home/filebrowser/data
    ports:
      - 7999:80
    restart: unless-stopped
```
> The Docker image looks for its config at `/home/filebrowser/data/config.yaml` by default, which is exactly where the volume mount above places it — so no extra environment variable is needed. The `:latest` tag includes FFmpeg for video/office thumbnails; swap to `:stable-slim` if you want the smaller core-only image without media previews.
{.is-info}

# 3 · Logging In
The default user is `admin` and the default password is `admin`.
> Change the admin password immediately after your first login (top-right user menu → **Settings**). Leaving it at the default exposes full admin access to anyone who can reach the page.
{.is-danger}

