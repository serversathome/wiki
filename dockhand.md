---
title: Dockhand
description: A guide to depoying Dockhand
published: true
date: 2026-06-22T13:30:02.111Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:20.950Z
---

# <img src="/dockhand.webp" class="tab-icon"> What is Dockhand?

**Dockhand** is a modern, self-hosted Docker management UI for running, deploying, and observing containers across one or many hosts. It covers real-time container control, a visual Compose stack editor, Git-based deployments, interactive shells, log streaming, image and CVE scanning, and a network graph — all from a single dashboard with no cloud dependency or telemetry.

A few things that set it apart from the usual Portainer/Dockge crowd:

- **Hardened base image** — Dockhand builds its own OS layer from scratch using [Wolfi](https://github.com/wolfi-dev/os) packages via apko, with every package explicitly declared. The goal is a near-zero-CVE image rather than shipping Alpine with whatever vulnerabilities come along for the ride.
- **Safe-pull auto-updates** — when checking for updates, new images are pulled to a temporary tag and scanned (Grype/Trivy) *before* touching your running container. If the scan exceeds your vulnerability criteria, the temp image is deleted and the running container is left alone.
- **Multi-host via the Hawser agent** — an open-source Go agent that makes **outbound** connections back to Dockhand, so you can manage hosts behind NAT, firewalls, or dynamic IPs with no inbound ports exposed.
- **GitOps** — point a stack at a Git repo and use webhooks or scheduled auto-sync to keep it deployed.


# 1 · Deploy Dockhand
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

```yaml
services:
  dockhand:
    image: fnsys/dockhand:v1.0.33
    container_name: dockhand
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
    ports:
      - "3000:3000"
    volumes:
      - /mnt/tank/configs/dockhand:/app/data
      - /var/run/docker.sock:/var/run/docker.sock
    restart: unless-stopped
```



## <img src="/truenas.png" class="tab-icon"> TrueNAS

Dockhand is in the **Community** train of the TrueNAS Apps catalog (maintained by iX), so you can install it from the UI:

1. Go to **Apps** → **Discover** and search for **Dockhand**.
2. Click **Install**.
3. Configure the key settings:
   - **Storage** — point the Dockhand data/config storage at a host path dataset, e.g. `/mnt/tank/configs/dockhand` (ix-volume also works, but a host path is easier to back up and inspect).
   - **Docker socket** — the app declares a host mount for `/var/run/docker.sock`; leave this enabled, it's how Dockhand talks to the engine.
   - **Web port** — defaults to the app's assigned port; change it if it collides with something else.
   - **Resources** — the memory limit defaults to 4096 MB; adjust to taste.
4. Click **Install** and wait for the app to report **Running**.
5. Open the **Web UI** from the app's card.



# 2 · First Run & Authentication

By default a fresh Dockhand instance starts with **authentication OFF** — anyone who can reach the port has full control of your Docker hosts. Lock it down before exposing it anywhere:

1. In the left sidebar, open **Settings** → **Authentication** → **Users**.
2. Click **Add User** and create your admin account (username + password).
3. Back on the Authentication screen, flip the **Authentication** toggle from **OFF** to **ON**.
4. Sign out and confirm you're prompted to log in.

> 
> Never expose a no-auth Dockhand instance to the internet or an untrusted LAN. Access to the Docker socket is effectively root on the host. Keep it behind your reverse proxy / Zero Trust and only enable it after auth is on.
{.is-danger}

For larger setups, Dockhand also supports **OIDC / SSO** and local users, with optional **RBAC** in the Enterprise tier. Configure OIDC under **Settings** → **Authentication**.

# 3 · Adding Environments (Multi-Host)

Dockhand manages multiple Docker hosts from one dashboard. Each host is an **Environment**.

1. Go to **Settings** → **Environments** → **+ Add environment**.
2. Choose how to connect:
   - **Unix socket** — for the local host Dockhand is running on (the socket you already mounted).
   - **Hawser agent** — for remote hosts (recommended for anything across the network or behind NAT).
   - **Direct TCP** — if you expose the Docker API over TCP yourself (use TLS).

## 3.1 Hawser Agent (Remote Hosts)

On each remote machine, install Docker and deploy the Hawser agent:

```yaml
services:
  hawser:
    image: ghcr.io/finsys/hawser:latest
    container_name: hawser
    environment:
      - HAWSER_NODE_NAME=remote-node-1   # optional friendly name
      # - HAWSER_TLS_ENABLED=true        # optional encrypted transport
    ports:
      - "2376:2376"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    restart: unless-stopped
```

Then back in Dockhand, add the environment using the remote host's IP, the agent port (`2376`), and the token. Once the connection test passes, the host appears in your dashboard alongside the others.

> 
> Hawser dials **out** to Dockhand, so the remote host needs no inbound firewall rules or port forwards — handy for NetBird-meshed or NAT'd nodes.
{.is-success}

