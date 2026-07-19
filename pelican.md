---
title: Pelican
description: A guide to installing Pelican Panel
published: true
date: 2026-07-19T10:43:22.971Z
tags: 
editor: markdown
dateCreated: 2026-07-17T17:28:21.752Z
---

# <img src="/pelican-panel.png" class="tab-icon"> What is Pelican Panel?

**Pelican Panel** is a modern, open-source game server management panel — a spiritual successor to Pterodactyl, built on Laravel/PHP. It gives you a clean web UI to deploy and manage game servers (Minecraft, Rust, Valheim, and hundreds more via "Eggs"), with user accounts, resource limits, file management, and live console access.

Pelican has two parts: the **Panel** (the web UI) and **Wings** (the daemon that runs the actual game servers via Docker). This guide runs **both as Docker stacks in Dockge** on TrueNAS — no VM or LXC required.

> 
> Pelican is still in **Beta**, and Docker is the maintainers' planned future install method (currently marked "work in progress"). It's stable enough for a homelab — just pin your working image tag and read the changelog before updating.
{.is-warning}



# 1 · Deploy the Panel


Create a new stack in Dockge called **`pelican`** and paste the compose below. It uses the official image with its built-in Caddy webserver and **SQLite**, storing everything under `/mnt/tank/configs/pelican` per the wiki's folder convention.

```yaml
services:
  pelican:
    image: ghcr.io/pelican-dev/panel:latest
    container_name: pelican
    environment:
      - XDG_DATA_HOME=/pelican-data
      - APP_URL=https://pelican.example.com 
      - APP_ENV=production
      - APP_DEBUG=false
      - BEHIND_PROXY=true
      - TRUSTED_PROXIES=172.20.0.0/16
      - LE_EMAIL=you@example.com
      - MAIL_DRIVER=log
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/pelican:/pelican-data
    ports:
      - 88:88
    restart: unless-stopped
```

Point your reverse proxy (Cloudflare Tunnel / DockFlare / etc.) at the container on **port 88**, and set `APP_URL` to match the hostname you expose.

> Set the `/mnt/tank/configs/pelican` dataset to **Generic** permissions and give other full permissions
{.is-success}


> **Just testing on the LAN?** Set `APP_URL=http://<truenas-ip>:8088`, use `8088:8088` for ports, and drop the `BEHIND_PROXY`/`TRUSTED_PROXIES` lines. Don't use `80:80`/`443:443` — they collide with the TrueNAS web UI.
{.is-info}

> Once its running go to `http://{IP}:port/installer` to setup the panel
{.is-success}



## 1.1 Grab the App Key & Run the Installer

Deploy the stack, then open the **panel** container logs in Dockge and copy the generated key line (`Generated app key: ...`).

> 
> **Back up that APP_KEY.** It encrypts all sensitive data (API keys, etc.). It lives under `/mnt/tank/configs/pelican`, but keep a copy off the box — lose it and encrypted data is **unrecoverable, even with database backups.**
{.is-danger}

Now finish setup in the browser at:

```
https://pelican.example.com/installer
```

Choose **SQLite** for the database and complete the guided steps (cache, sessions, email). The installer creates your first **admin user** at the end. 

# <img src="/docker.png" class="tab-icon"> 2 · Deploy Wings

Wings needs a node config **before** it starts cleanly, so do this in order.

## 2.1 Create the Node in the Panel

In the Panel: **Admin → Nodes → Create Node**. Set the **FQDN** to the TrueNAS host's LAN IP (or a domain pointing at it), and match the SSL setting to your Panel — if the Panel is HTTPS, the node should be too. After saving, open the node's **Configuration** tab and copy the generated YAML.

On the TrueNAS host, create the config dir and save that YAML into it — this is the `etc` bind from the stack below:

```bash
mkdir -p /mnt/tank/configs/wings/{etc,data,logs,tmp}
chown -R 568:568 /mnt/tank/configs/wings
```
Paste the node config into `/mnt/tank/configs/wings/etc/config.yml`

> 
> In that `config.yml`, point Wings' data directory at the 1:1 mount so server files land under the wiki path. Set `system.data` (and let `root_directory` follow) to `/mnt/tank/configs/wings/data`, and set the tmp/backup dir to `/mnt/tank/configs/wings/tmp`. The Panel also exposes the **Daemon Server File Directory** field on the node — set it to `/mnt/tank/configs/wings/data/volumes` so the generated config matches.
{.is-info}

## 2.2 Wings Stack

Create a second Dockge stack called **`wings`** with this compose. UID/GID are set to **568** to match the rest of the lab.

```yaml
services:
  wings:
    image: ghcr.io/pelican-dev/wings:latest
    container_name: wings
    restart: unless-stopped
    tty: true
    environment:
      - TZ=America/New_York
      - WINGS_UID=568
      - WINGS_GID=568
      - WINGS_USERNAME=pelican
    ports:
      - 87:87     # Wings API — the Panel talks to this
      - 2022:2022     # SFTP
    volumes:
      - /mnt/tank/configs/wings/etc:/etc/pelican
      - /mnt/tank/configs/wings/data:/mnt/tank/configs/wings/data  # MUST be 1:1
      - /mnt/tank/configs/wings/logs:/var/log/pelican
      - /mnt/tank/configs/wings/tmp:/mnt/tank/configs/wings/tmp    # MUST be 1:1
      # --- Host system paths — do NOT rename these ---
      - /var/run/docker.sock:/var/run/docker.sock
      - /etc/ssl/certs:/etc/ssl/certs:ro
```

> 
> **Path parity — the one that bites people.** Wings drives the *host* Docker daemon through the socket, so any path it hands to that daemon must resolve to the **same absolute path** on the host and inside the container. That's why `data` and `tmp` above are mounted 1:1. You can name that path whatever you like — it just has to be identical on both sides, and `config.yml` has to point at it. The `etc` and `logs` mounts are Wings-internal, so those can map to the standard container paths.
{.is-danger}


Deploy the stack. Check the **wings** container logs in Dockge — it should connect to the Panel with no errors.

# 3 · Allocations & First Server

Back in the Panel, open your node and add **Allocations** (the host IP plus the ports your servers will use). Then create a server, pick an Egg, assign an allocation, and Wings spins it up as its own container.

> 
> Each game server publishes its own ports on the TrueNAS host (they're sibling containers). Reserve a port range and watch for collisions with your other apps.
{.is-info}

