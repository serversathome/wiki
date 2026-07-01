---
title: Tugtainer
description: A guide to deploying Tugtainer
published: true
date: 2026-07-01T01:23:13.149Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:09:08.384Z
---

# <img src="/tugtainer.png" class="tab-icon"> What is Tugtainer?

**Tugtainer** is a self-hosted app that automates updates for your Docker containers through a clean web UI. Think of it as a Watchtower-style update checker, but with a full dashboard, per-container control, multi-host management, and a proper dependency-aware update engine.

It checks your running containers against their registries, flags the ones with a newer image, and can either notify you or pull, recreate, and restart them automatically on a cron schedule. You decide per container whether it's **check-only** (notify me) or **auto-update** (do it for me), so nothing moves unless you ask it to.

Key things that set it apart from the usual auto-updaters:

- **Web UI with auth** — password, OIDC, or no auth at all, your call
- **Multiple hosts** from a single dashboard via a lightweight agent
- **Socket-proxy support** so you never have to mount the raw Docker socket
- **Dependency-aware updates** that respect Compose `depends_on` (and custom links)
- **Separate check and update schedules** with independent crontabs
- **Notifications** to ~100 services through Apprise, with Jinja2 templating
- **Public API endpoints** you can wire into a dashboard or status page
- **Image pruning**, container inspect/logs, and basic start/stop control

> Tugtainer is distributed as-is and the author does **not** recommend it for production use. Automatic updates are **disabled by default** — you opt in per container. Keep regular backups of anything important before letting it recreate containers.
{.is-warning}

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Tugtainer

The project now ships with a **socket-proxy by default**, so Tugtainer talks to Docker over TCP instead of touching the raw socket. Pick the tab that fits how you like to run things.

# {.tabset}


## Direct Socket

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



## Socket Proxy

```yaml
networks:
  tugtainer:
    driver: bridge

services:
  socket-proxy:
    image: lscr.io/linuxserver/socket-proxy:latest
    container_name: tugtainer-socket-proxy
    environment:
      CONTAINERS: 1
      EVENTS: 1
      IMAGES: 1
      INFO: 1
      NETWORKS: 1
      PING: 1
      POST: 1
      VERSION: 1
      LOG_LEVEL: warning
      TZ: America/New_York
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    read_only: true
    tmpfs:
      - /run
    networks:
      - tugtainer
    restart: unless-stopped
    labels:
      dev.quenary.tugtainer.protected: "true"

  tugtainer:
    image: ghcr.io/quenary/tugtainer:1
    container_name: tugtainer
    depends_on:
      - socket-proxy
    environment:
      AGENT_SECRET: CHANGE_ME!
      DOCKER_HOST: tcp://socket-proxy:2375
      TZ: America/New_York
    volumes:
      - /mnt/tank/configs/tugtainer:/tugtainer
    networks:
      - tugtainer
    ports:
      - "9412:80"
    restart: unless-stopped
    labels:
      dev.quenary.tugtainer.protected: "true"
```

- `CONTAINERS`, `IMAGES`, `POST`, `INFO`, `PING` are the minimum for the **check** feature; `NETWORKS` is needed for the **update** feature.
- `DOCKER_HOST` points Tugtainer at the proxy over the Compose network. The service name `socket-proxy` resolves even though the container is named `tugtainer-socket-proxy`, so it won't collide with a socket-proxy in another stack.

> The proxy container is marked `protected` so Tugtainer can't try to update the thing it depends on to reach Docker.
{.is-info}

# 2 · How Tugtainer Works

Scheduling is cron-based, and you can set **independent schedules for checks and updates** in the app settings — for example, check hourly but only apply updates in the middle of the night.

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

| Label                                         | Effect                                                                                                                                               |
| --------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `dev.quenary.tugtainer.protected=true`        | Container can't be stopped/updated by the app. Used for Tugtainer itself, the agent, and the socket-proxy. Check-only still works for notifications. |
| `dev.quenary.tugtainer.depends_on="db,cache"` | Declares a dependency on other containers by name, even outside the same Compose project.                                                            |
{.dense}

> You **cannot** update an **agent** or a **socket-proxy** from inside Tugtainer — they're how it talks to Docker. Don't put them in a Compose project alongside containers you auto-update, or the update will error. Mark them `protected` and recreate them manually (or with Portainer/Dockge).
{.is-warning}

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
- The agent supports the same socket-proxy approach — drop the socket mount and set `DOCKER_HOST: tcp://socket-proxy:2375` instead.

# 5 · Authentication

Tugtainer uses **password auth by default**. The password is stored as an encrypted hash on disk, and auth is configured entirely through environment variables.

## 5.1 Password (default)

Nothing to configure — set a password on first login. A couple of env vars are worth setting anyway:

- **`JWT_SECRET_KEY`** — set this to a fixed value. Otherwise it's regenerated on every restart and you're logged out each time the container bounces.
- **`HTTPS=true`** — enables strict, HTTPS-only cookies. Set this when you front Tugtainer with a reverse proxy or tunnel.
- **`DOMAIN`** — locks auth to a single domain.

## 5.2 OIDC

Since **v1.6.0** you can authenticate against an OpenID Connect provider (Authentik, Authelia, Keycloak, Google, etc.) instead of, or alongside, the password.

| Variable              | Purpose                                                                       |
| --------------------- | ----------------------------------------------------------------------------- |
| `OIDC_ENABLED`        | Set `true` to turn on OIDC.                                                    |
| `OIDC_WELL_KNOWN_URL` | Provider's discovery endpoint, e.g. `https://auth.example.com/.well-known/openid-configuration`. |
| `OIDC_CLIENT_ID`      | Client ID from your provider.                                                 |
| `OIDC_CLIENT_SECRET`  | Client secret from your provider.                                             |
| `OIDC_REDIRECT_URI`   | Must match the provider, e.g. `https://tugtainer.example.com/api/auth/oidc/callback`. |
| `OIDC_SCOPES`         | Space-separated scopes. Default `openid profile email`.                       |
{.dense}

Set **`DISABLE_PASSWORD=true`** if you want OIDC to be the *only* way in.

## 5.3 Disabling Auth

Since **v1.16.0** you can turn auth off entirely with **`DISABLE_AUTH=true`**. Only do this if something in front of Tugtainer is already handling access — for example a Cloudflare Access / Zero Trust policy or an authenticating reverse proxy. Never expose an auth-disabled instance directly.

# 6 · Private Registries

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

The Docker CLI inside the container handles authentication from there. You can also point at a non-default location with the **`DOCKER_CONFIG`** env var.

# 7 · Environment Variables

None are required, but these are the ones you'll actually reach for. The full list lives in the project's `.env.example`.

| Variable            | Applies to | Default            | Purpose                                                                    |
| ------------------- | ---------- | ------------------ | -------------------------------------------------------------------------- |
| `TZ`                | app, agent | UTC                | Timezone for schedules and logs.                                           |
| `LOG_LEVEL`         | app, agent | `warning`          | `critical`, `error`, `warning`, `info`, `debug`, `trace`.                  |
| `AGENT_ENABLED`     | app        | `true`             | Set `false` to disable the built-in agent (e.g. host with no socket access). |
| `AGENT_SECRET`      | app, agent | empty              | Signs backend↔agent requests. Also set it per host in the UI.              |
| `DOCKER_HOST`       | app, agent | empty              | Point at a socket-proxy, e.g. `tcp://socket-proxy:2375`.                    |
| `DOCKER_CONFIG`     | app, agent | `~/.docker`        | Location of Docker client config (private registries).                     |
| `DISABLE_AUTH`      | app        | `false`            | Turn off authentication entirely (front it with something else).           |
| `DISABLE_PASSWORD`  | app        | `false`            | OIDC-only login.                                                           |
| `JWT_SECRET_KEY`    | app        | autogenerated      | Set a fixed value so sessions survive restarts.                            |
| `HTTPS`             | app        | `false`            | Strict, HTTPS-only cookies. Enable behind a proxy/tunnel.                  |
| `DOMAIN`            | app        | empty              | Restrict auth to a single domain.                                          |
| `ENABLE_PUBLIC_API` | app        | `false`            | Expose the unauthenticated public API endpoints (see below).              |
| `GH_TOKEN`          | app        | empty              | GitHub token to dodge rate limits on Tugtainer's own update check.         |
| `DOCKER_TIMEOUT`    | agent      | `15`               | CLI timeout (seconds) for fast ops like inspect.                           |
{.dense}


# <img src="/youtube.png" class="tab-icon"> 8 · Video

https://youtu.be/PTjco8Fryqg