---
title: Arcane
description: A guide to deploying Arcane in docker
published: true
date: 2026-06-13T18:55:58.602Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:03:09.587Z
---

# <img src="/arcane.png" class="tab-icon"> What is Arcane?

**Arcane** is a modern, self-hosted web UI for managing **Docker** and **Docker Swarm** — think containers, images, volumes, networks, ports, and Compose projects all from one place, plus a typed REST API and a companion CLI. It goes well beyond a basic dashboard: multi-host management via agents, image-update detection with optional auto-update/auto-heal, Trivy vulnerability scanning, GitOps, notifications, and full RBAC with OIDC SSO.


# 1 · Deploy Arcane
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

Arcane needs **two secrets** before it'll start: an encryption key (for secrets at rest) and a JWT signing secret (for login tokens). Generate them first:

```bash
openssl rand -base64 32   # use the output for ENCRYPTION_KEY
openssl rand -base64 32   # run again for JWT_SECRET
```

Then deploy via Dockge:

```yaml
services:
  arcane:
    image: ghcr.io/getarcaneapp/manager:latest
    container_name: arcane
    ports:
      - "3552:3552"
    environment:
      - ENCRYPTION_KEY=PASTE_FIRST_OPENSSL_VALUE_HERE
      - JWT_SECRET=PASTE_SECOND_OPENSSL_VALUE_HERE
      - PROJECTS_DIRECTORY=/mnt/tank/stacks   # where your Compose projects live
      - PUID=568
      - PGID=568
      - TZ=America/New_York
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock   # manage the host's Docker
      - /mnt/tank/configs/arcane:/app/data          # persistent app data (DB, etc.)
      - /mnt/tank/stacks:/mnt/tank/stacks      # must match PROJECTS_DIRECTORY
    cgroup: host                       # helps Arcane detect its own container
    healthcheck:
      test: ["CMD", "./arcane", "health", "--timeout", "2s"]
      interval: 10s
      timeout: 3s
      retries: 5
      start_period: 15s
    restart: unless-stopped
```

`PROJECTS_DIRECTORY` is the folder Arcane scans for your Compose projects — point it at your existing stacks directory and mount that **same path** into the container.



## <img src="/truenas.png" class="tab-icon"> TrueNAS

Arcane is in the TrueNAS **Community** train.

1. Navigate to **Apps** in the TrueNAS UI and click **Discover Apps**.
2. Search for **Arcane** and click **Install**.
3. Configure the key settings:
   - **Encryption Key** — paste a value from `openssl rand -base64 32`.
   - **JWT Secret** — paste a second `openssl rand -base64 32` value.
   - **Host Path (App Data)**: `/mnt/tank/configs/arcane`
   - **Projects Directory**: point at your stacks path (e.g. `/mnt/bigdeal/stacks`) and add a matching host-path mount.
   - **Docker Socket**: the app mounts `/var/run/docker.sock` automatically.
4. The app runs as user/group **568** (the `apps` group) by default.
5. Click **Install** and wait for the **Running** state.



# 2 · First Login & Securing the Admin Account

Arcane seeds a **default administrator** on first run:

| Field | Value |
|-------|-------|
| Username | `arcane` |
| Password | `arcane-admin` |


This account is flagged **"requires password change"**, so Arcane forces you to set a new password immediately after the first login.






# 3 · Running Behind Cloudflare Tunnel / Reverse Proxy

For external access, terminate TLS at your reverse proxy (or Cloudflare Tunnel) and forward to port `3552`. Two settings matter:

- **`APP_URL`** — your public URL (e.g. `https://arcane.serversatho.me`). Used for generated links and OIDC redirects.
- **`TRUSTED_PROXIES`** — the CIDR(s) of your proxy so Arcane honors `X-Forwarded-*` headers for client IP and HTTPS detection.

```yaml
    environment:
      - ENCRYPTION_KEY=...
      - JWT_SECRET=...
      - APP_URL=https://arcane.serversatho.me
      - TRUSTED_PROXIES=172.16.0.0/12      # your Docker/proxy subnet
```

> 
> Arcane can also terminate TLS itself (HTTP/2 over TLS) without a proxy by setting `TLS_ENABLED=true`, `TLS_CERT_FILE`, and `TLS_KEY_FILE`.
{.is-info}

# 4 · Adding Remote Docker Hosts (Agents)

To manage Docker hosts other than the one Arcane runs on, deploy an **agent** on each remote host. Pick the mode based on your network:

| Mode | Use when | How it connects |
|------|----------|-----------------|
| **Edge agent** | The remote host is behind NAT/firewall, or you don't want inbound ports. | Dials **out** to the manager over a tunnel (WebSocket / gRPC / poll), optionally with mTLS. |
| **Direct agent** | You have a clear path from manager → host (LAN, VPN, NetBird mesh). | Listens **passively on TCP 3553**; the manager connects in. |


In the manager UI go to **Environments → add a new environment / agent**, generate an **agent token**, and note your manager's URL. Then deploy the agent on the remote host.

**Edge agent** (no inbound ports needed):

```yaml
services:
  arcane-edge-agent:
    image: ghcr.io/getarcaneapp/agent:latest
    container_name: arcane-edge-agent
    environment:
      - EDGE_AGENT=true
      - AGENT_TOKEN=arc_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
      - MANAGER_API_URL=https://arcane.serversatho.me
      - EDGE_TRANSPORT=websocket          # or: grpc
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /mnt/tank/configs/arcane-agent:/app/data
    cgroup: host
    restart: unless-stopped
```

**Direct agent** (exposes port 3553):

```yaml
services:
  arcane-agent:
    image: ghcr.io/getarcaneapp/agent:latest
    container_name: arcane-agent
    ports:
      - "3553:3553"
    environment:
      - AGENT_MODE=true
      - AGENT_TOKEN=arc_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
      - MANAGER_API_URL=http://10.99.0.200:3552
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /mnt/tank/configs/arcane-agent:/app/data
    cgroup: host
    restart: unless-stopped
```

The new environment appears in the manager and becomes selectable in the switcher. For edge agents you can enforce **mutual TLS** with the `EDGE_MTLS_*` variables (the manager can act as the CA).

# 5 · Updates, Scanning & Notifications

## 5.1 Image Updates, Auto-Update & Auto-Heal

Arcane detects when a newer image digest is available for your running containers:

1. **Polling** — a background job periodically checks images against their registries and records what has updates. Review them under **Updates**.
2. **Manual update** — apply an available update from the UI with one click.
3. **Auto-update** — optionally apply updates automatically on a schedule.
4. **Auto-heal** — optionally recover containers that become unhealthy.

Tune or disable these under **Settings → Jobs/Schedules**, or turn off update checking entirely with `UPDATE_CHECK_DISABLED=true`.

## 5.2 Vulnerability Scanning (Trivy)

Arcane integrates **Trivy** to scan images for known vulnerabilities. Scans run as ephemeral scanner containers with full lifecycle tracking (phases, results, ignore filters, summaries, notifications). Trigger scans from the **Images** view or schedule them.

On hardened hosts you may need:

| Variable | When |
|----------|------|
| `TRIVY_SECURITY_OPTS=label=disable` | SELinux systems. |
| `TRIVY_PRIVILEGED=true` | Only if security options alone aren't enough. |
| `TRIVY_SCAN_TIMEOUT` | Large images. |
{.dense}

## 5.3 Notifications

Configure outbound alerts under **Settings → Notifications** for update-available, vulnerability, prune, auto-heal, and test events. Supported channels: **Discord, Slack, Telegram, Email (SMTP), Gotify, Ntfy, Pushover, generic Webhooks**, and the **Shoutrrr** meta-provider.

> 
> Use the **"send test"** action after configuring a provider to confirm delivery before relying on it.
{.is-success}

# 6 · Single Sign-On (OIDC)

To let users log in with your identity provider (Authentik, Keycloak, Google, etc.), enable OIDC via environment variables (and/or **Settings → OIDC**):

```yaml
    environment:
      - OIDC_ENABLED=true
      - OIDC_ISSUER_URL=https://idp.serversatho.me/realms/main
      - OIDC_CLIENT_ID=arcane
      - OIDC_CLIENT_SECRET=...
      - OIDC_SCOPES=openid profile email groups
      - OIDC_GROUPS_CLAIM=groups
      - OIDC_PROVIDER_NAME=Servers@Home SSO
      - OIDC_ROLE_MAPPINGS=...                  # map IdP groups → Arcane roles
      - OIDC_AUTO_REDIRECT_TO_PROVIDER=true     # send users straight to the provider
```

Set your provider's redirect/callback URL to your Arcane `APP_URL`. Admin status is determined by Arcane's permission model — not by IdP role names alone.



# <img src="/youtube.png" class="tab-icon"> 7 · Video
