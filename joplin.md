---
title: Joplin
description: A guide to deploy Joplin server
published: true
date: 2026-01-27T12:16:53.162Z
tags: 
editor: markdown
dateCreated: 2026-01-27T12:16:53.162Z
---

# <img src="/joplin.png" class="tab-icon"> What is Joplin?

**Joplin** is an open source note-taking and to-do application with sync capabilities. Joplin Server allows you to self-host your own sync backend, giving you full control over your notes across all your devices.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Joplin Server

```yaml
services:
  joplin:
    image: joplin/server:latest
    container_name: joplin
    restart: unless-stopped
    ports:
      - "22300:22300"
    environment:
      - APP_PORT=22300
      - APP_BASE_URL=http://YOUR_SERVER_IP:22300
      - DB_CLIENT=pg
      - POSTGRES_HOST=joplin-db
      - POSTGRES_PORT=5432
      - POSTGRES_DATABASE=joplin
      - POSTGRES_USER=joplin
      - POSTGRES_PASSWORD=joplinrocks
    depends_on:
      - joplin-db

  joplin-db:
    image: postgres:16
    container_name: joplin-db
    restart: unless-stopped
    environment:
      - POSTGRES_DB=joplin
      - POSTGRES_USER=joplin
      - POSTGRES_PASSWORD=joplinrocks
    volumes:
      - /mnt/tank/configs/joplin:/var/lib/postgresql/data
```

1. Change all the default passwords. Make sure when you change them in the joplin service you mirror those changes in the joplin-db service.
2. Replace `YOUR_SERVER_IP` with your server's IP address or domain name.

> 
> If using a reverse proxy, set `APP_BASE_URL` to your full domain like `https://joplin.yourdomain.com`
{.is-info}

# 2 · First Login

After deployment, access the web interface at `http://YOUR_SERVER_IP:22300`.

| Default Email | Default Password |
|---|---|
| `admin@localhost` | `admin` |

> 
> Change these credentials immediately after first login!
{.is-warning}

# 3 · Creating Users

1. Log in with the default admin credentials
2. Navigate to **Users** in the admin panel
3. Click **Add User**
4. Enter email and password for each user

> 
> Each person syncing notes needs their own user account. Don't share the admin account.
{.is-info}

# 4 · Client Setup

## 4.1 Desktop (Windows/Mac/Linux)

1. Download Joplin from [joplinapp.org](https://joplinapp.org/download/)
2. Open Joplin → **Tools** → **Options** → **Synchronisation**
3. Set **Synchronisation target** to **Joplin Server**
4. Enter:
   - **Joplin Server URL:** `http://YOUR_SERVER_IP:22300`
   - **Email:** your user email
   - **Password:** your user password
5. Click **Check synchronisation configuration**
6. Click **Apply**

## 4.2 Mobile (iOS/Android)

1. Install Joplin from the App Store or Play Store
2. Open the app → **Configuration** → **Synchronisation**
3. Select **Joplin Server**
4. Enter the same credentials as desktop
5. Tap **Check synchronisation configuration**


# 5 · Troubleshooting

## Cannot connect from client
- Verify `APP_BASE_URL` matches exactly how you're accessing the server
- Check that port `22300` is accessible from your client device
- Ensure you're using your user credentials, not the admin credentials

## Sync conflicts
Joplin handles conflicts by creating duplicate notes with a conflict marker. Review and merge these manually in the desktop client.
