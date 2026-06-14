---
title: Tugtainer
description: A guide to deploying Tugtainer
published: true
date: 2026-06-14T11:00:57.900Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:09:08.384Z
---

# <img src="/tugtainer.png" class="tab-icon"> What is Tugtainer?

**Tugtainer** is a self-hosted app that automates updates for your Docker containers through a clean web UI. Think of it as a Watchtower-style update checker, but with a full dashboard, per-container control, multi-host management, and a proper dependency-aware update engine.

It checks your running containers against their registries, flags the ones with a newer image, and can either notify you or pull, recreate, and restart them automatically on a cron schedule. You decide per container whether it's **check-only** (notify me) or **auto-update** (do it for me), so nothing moves unless you ask it to.

Key things that set it apart from the usual auto-updaters:

- **Web UI with auth** instead of a CLI/env-only config
- **Multiple hosts** from a single dashboard via a lightweight agent
- **Socket-proxy support** so you never have to mount the raw Docker socket
- **Dependency-aware updates** that respect Compose `depends_on` (and custom links)
- **Notifications** to ~100 services through Apprise, with Jinja2 templating
- **Image pruning**, container inspect/logs, and basic start/stop control

> 
> Tugtainer is distributed as-is and the author does **not** recommend it for production use. Automatic updates are **disabled by default** — you opt in per container. Keep regular backups of anything important before letting it recreate containers.
{.is-warning}

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Tugtainer


```yaml
services:
  tugtainer:
    image: ghcr.io/quenary/tugtainer:1
    container_name: tugtainer
    environment:
      AGENT_SECRET: CHANGE_ME!
      TZ: America/New_York
    volumes:
      - /mnt/tank/configs/tugtainer:/tugtainer
      - /var/run/docker.sock:/var/run/docker.sock:ro
    ports:
      - "9412:80"
    restart: unless-stopped
    labels:
      dev.quenary.tugtainer.protected: "true"
```





# 2 · How Tugtainer Works

## 2.1 Check Process

A scheduled or manual **check** does the following per container:

1. Confirms the container is eligible (skips containers built from a purely local image with no registry).
2. Optionally pulls the image (off by default — handy if you front a registry proxy).
3. Requests the current digest of the image from its registry.
4. Compares the local digest against the remote digest.
5. If they differ, the container is **marked as available**.

A **scheduled** check runs against every enabled host and every container you've selected for **auto-check**. A **manual** check runs against *all* containers regardless of the auto-check toggle (or just one, if you click a single container).

## 2.2 Update Process

Updates are **dependency-aware** so linked containers come up and down in the right order:

- All containers on a host are processed as one set, and a global dependency graph is built.
- **Protected** containers are skipped; non-running containers are skipped by default (configurable).
- Tugtainer calculates which containers are **updatable** (marked available + selected for auto-update, or clicked), then which are **affected** (anything that depends on an updatable container).
- A topological order is built. Updatable images are pulled, all involved containers are stopped from most-dependent to least, then started back up in reverse — recreating updatable ones with the new image and restarting affected ones.

If a recreate fails, Tugtainer tries to **roll back** to the old image and reports the result.



# 3 · Custom Labels

Two labels control how Tugtainer treats a container. Add them under `labels:` in your Compose.

| Label | Effect |
|-------|--------|
| `dev.quenary.tugtainer.protected=true` | Container can't be stopped/updated by the app. Used for Tugtainer itself, the agent, and the socket-proxy. Check-only still works for notifications. |
| `dev.quenary.tugtainer.depends_on="db,cache"` | Declares a dependency on other containers by name, even outside the same Compose project. |


> 
> You **cannot** update an **agent** or a **socket-proxy** from inside Tugtainer — they're how it talks to Docker. Don't put them in a Compose project alongside containers you auto-update, or the update will error. Mark them `protected` and recreate them manually (or with Portainer/Dockge).
{.is-danger}

# 4 · Remote Hosts (Agent)

To manage other machines from one UI, deploy the **Tugtainer Agent** on each remote host, then add it under **Menu → Hosts** in the UI.

```yaml
services:
  tugtainer-agent:
    image: ghcr.io/quenary/tugtainer-agent:1
    container_name: tugtainer-agent
    environment:
      AGENT_SECRET: CHANGE_ME!
      TZ: America/New_York
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    ports:
      - "9413:8001"
    restart: unless-stopped
    labels:
      dev.quenary.tugtainer.protected: "true"
```

- The remote machine must be reachable from the primary instance.
- The backend and agent talk over **HTTP** — put a reverse proxy in front for HTTPS.
- **AGENT_SECRET** signs backend↔agent requests; set it per host in the UI (even for the local agent).

# 5 · Private Registries

To check/update images from a private registry, mount your Docker config into the Tugtainer (or agent) container that runs the private image.

1. Create the config on the host — either `docker login <registry>` or by hand:

```json
{
  "auths": {
    "<registry>": {
      "auth": "base64 encoded 'username:password_or_token'"
    }
  }
}
```

2. Mount it read-only into the container:

```yaml
    volumes:
      - $HOME/.docker/config.json:/root/.docker/config.json:ro
```

The Docker CLI inside the container handles authentication from there.


