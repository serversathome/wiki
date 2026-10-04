---
title: Dozzle
description: A guide to deploying Dozzle on TrueNAS Scale and via Docker Compose
published: true
date: 2026-10-04T11:42:37.490Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:29.434Z
---

# <img src="/dozzle.png" class="tab-icon"> What is Dozzle?

**Dozzle** is a lightweight, real-time log viewer for Docker, Swarm and Kubernetes. It streams logs from every container straight to your browser, merges several containers side by side, parses JSON and structured logs into fields, and shows live CPU, memory and network charts right next to the logs. It is a single binary with no database, and your logs never leave your network.

Version 11 is the biggest update Dozzle has ever had: a completely redesigned interface, a first-run setup wizard, sign in with GitHub or any OIDC provider (Pocket ID, Authentik, Keycloak, Google), image update checks with one-click and scheduled updates, app icons, host metrics, and a built-in MCP server so AI assistants can read your logs.

<div class="glance">
  <div><span>Port</span><b><code>8888</code></b></div>
  <div><span>Deploy via</span><b>TrueNAS app or Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty intermediate">Intermediate</b></div>
  <div class="glance-links">
    <a href="https://github.com/amir20/dozzle"><i class="mdi mdi-github"></i>Project</a>
    <a href="https://dozzle.dev"><i class="mdi mdi-book-open-variant"></i>Docs</a>
    <a href="https://apps.truenas.com/catalog/dozzle_community/"><i class="mdi mdi-server"></i>TrueNAS app</a>
    <a href="https://youtu.be/LNDkGBOfv6Y"><i class="mdi mdi-youtube"></i>Video walkthrough</a>
  </div>
</div>

> **Coming from the old version of this guide?** Dozzle used to need nothing but the Docker socket. It now keeps users, settings and its session key in `/data`, so add the `/data` volume below before you upgrade. Everyone gets signed out once after moving to v11.
{.is-info}

# 1 · Deploy Dozzle
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

Create the config folder and hand it to the apps user first:

```bash
mkdir -p /mnt/tank/configs/dozzle
chown -R 568:568 /mnt/tank/configs/dozzle
```

```yaml
services:
  dozzle:
    image: amir20/dozzle:latest
    container_name: dozzle
    user: "568:568"
    group_add:
      - "999"
    environment:
      - TZ=America/New_York
    ports:
      - "8888:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /mnt/tank/configs/dozzle:/data
      - /proc:/host/proc:ro
    restart: unless-stopped
```

1. Change `TZ` to your timezone. Dozzle uses it for the auto-update schedule.
2. Deploy the stack and open `http://your-server-ip:8888`.
3. The setup wizard opens on first launch. Continue to section 2.

> Because Dozzle runs as `568:568` instead of root, it needs to join the group that owns the Docker socket. That group is `999` on a standard TrueNAS install. Confirm yours with `stat -c '%g' /var/run/docker.sock` and put that number under `group_add`. If Dozzle shows no containers, this is the reason.
{.is-warning}

> The `/proc:/host/proc:ro` line is optional. It lets Dozzle show the host's uptime and load average on the host card (see 3.5). Leave it out and those two read-outs simply stay hidden.
{.is-info}

## <img src="/truenas.png" class="tab-icon"> TrueNAS

Create the dataset or folder `/mnt/tank/configs/dozzle` and set its owner to `apps` (568:568) before you start.

1. Navigate to **Apps** in the TrueNAS UI and click **Discover Apps**
2. Search for "Dozzle" and click **Install**
3. Configure the following settings:
   - **Timezone**: your local timezone
   - **Mount Docker Socket**: leave checked
   - **User ID / Group ID**: leave at `568` / `568`
   - **WebUI Port**: change the default `30064` to `8888`
   - **Additional Storage**: click **Add**
     - **Type**: Host Path
     - **Mount Path**: `/data`
     - **Host Path**: `/mnt/tank/configs/dozzle`
4. Optional, for host uptime and load: add a second **Additional Storage** entry with **Type** Host Path, **Mount Path** `/host/proc`, **Host Path** `/proc`, and **Read Only** checked. If TrueNAS refuses a path outside your pools, skip this step.
5. Click **Install**
6. Open `http://your-truenas-ip:8888` and the setup wizard will greet you. Continue to section 2.

> The TrueNAS app does not create a `/data` volume on its own. Without the Additional Storage entry above, your login, users and wizard settings are wiped every time the app updates.
{.is-warning}

> Let TrueNAS update the Dozzle app itself. Leave Dozzle's **Auto-update** set to **Off** in the wizard, and do not use Dozzle's **Update** action on TrueNAS catalog apps. Dozzle would recreate those containers behind the TrueNAS middleware's back. Dozzle updates are safe for your Dockge stacks.
{.is-info}

# 2 · First Run: the Setup Wizard

A fresh install now opens a short setup wizard. Everything it saves goes into `/data/dozzle.yml`, and you can reopen it later from **Settings**. Any setting you also pass as an environment variable wins over the wizard and shows up locked.

## 2.1 Login

Login comes first so nothing else can be changed on an open instance. The wizard checks that `/data` is persisted before it lets you continue. Pick one:

- **Dozzle account** creates a single user with a username and password and turns on the `simple` auth provider.
- **My proxy** is for Authelia, Authentik, Cloudflare Access and similar. Dozzle trusts the `Remote-User` header, so publish only the proxy and never Dozzle's own port.
- **OIDC** shows the environment variables you need to add yourself (see 3.2).
- **Continue without login** is fine if Dozzle is only reachable on your own network.

Dozzle restarts itself after you save an account, and the wizard continues once you sign in.

> Without login, these settings can only be changed within 15 minutes of the very first start of a new install. After that, use environment variables or turn login on.
{.is-info}

## 2.2 Actions and Shell

- **Start, stop and restart** turns on container actions, which also unlocks remove and update.
- **Shell** lets you attach to containers and run commands inside them from the browser.

> Shell access to a container is often as good as access to the host. Only turn it on if you need it, and never without login.
{.is-danger}

## 2.3 Hosts

Add other Docker machines running a Dozzle agent. The wizard tests the connection before saving, and new hosts appear in the sidebar without a restart. See section 4 for the agent compose file.

## 2.4 Dozzle Cloud

Optional hosted add-on for alerts, a daily summary, an AI assistant and long-term log search. Click **Not now** to skip it. Everything in this guide works without it.

## 2.5 Auto-update

Choose **Off**, **Daily** or **Weekly** and a time of day. At that time Dozzle checks for a newer image and, only if one exists, updates itself through a short-lived helper container. If the new version fails to start, the helper rolls back to the old one. This step needs actions turned on in 2.2.

## 2.6 Restart

The last step lists pending changes. Click **Restart Dozzle** and the page reloads on the new settings.

# 3 · What's New

## 3.1 The New Interface

Almost every screen was redrawn with a flatter, quieter look where color only shows up for things that need attention:

- The sidebar groups containers into collapsible sections with app icons, and status sits as a small badge on each icon.
- Warning and error lines carry a light tint so they stand out while scrolling.
- A floating readout shows where you are in a container's log history while you scroll.
- Pinned columns live in the URL, so a side-by-side view is a link you can send someone.
- OpenTelemetry and Pino log levels are now recognized automatically.

## 3.2 Sign in with GitHub or OIDC

Users can now sign in with GitHub or any OIDC provider. This sits on top of the `simple` provider, so `users.yml` stays the allowlist: an account that is not listed there cannot get in, and nothing is ever created automatically. Password login keeps working next to it.

Register Dozzle with your provider as a confidential client using this redirect URI:

```
https://dozzle.yourdomain.com/api/auth/callback
```

Then add the variables to your compose file. This example uses Pocket ID:

```yaml
    environment:
      - DOZZLE_AUTH_PROVIDER=simple
      - DOZZLE_AUTH_OIDC_ISSUER=https://id.yourdomain.com
      - DOZZLE_AUTH_OIDC_CLIENT_ID=dozzle
      - DOZZLE_AUTH_OIDC_CLIENT_SECRET=your-client-secret
      - DOZZLE_AUTH_OIDC_NAME=Pocket ID
```

OIDC matches users on their **verified email**, so make sure the `email` field in `/mnt/tank/configs/dozzle/users.yml` matches the address in your provider:

```yaml
users:
  admin:
    email: you@yourdomain.com
    name: Admin
    password: $2a$11$...
```

For GitHub, use `DOZZLE_AUTH_GITHUB_CLIENT_ID` and `DOZZLE_AUTH_GITHUB_CLIENT_SECRET` instead, and link users with a `github: yourhandle` line.

> Keep a password on at least one account. If nobody in `users.yml` has one, the login form disappears and a broken OAuth app locks everyone out until you edit the file by hand.
{.is-warning}

> Behind a reverse proxy, the proxy must send `X-Forwarded-Proto: https`, or Dozzle builds an `http://` callback that your provider rejects. If you would rather have the provider own the user list and roles with no `users.yml` at all, use `DOZZLE_AUTH_PROVIDER=oidc` instead.
{.is-info}

## 3.3 Update Checks and One-Click Updates

Dozzle now compares the image each container is running against what its registry serves. When they differ, a dot appears on the container's menu. The check uses lightweight manifest requests that do not count against Docker Hub pull limits.

With actions on, the dashboard shows an **N updates** button that opens a drawer listing every out-of-date container. Untick what you want to leave alone and press **Update**. Hosts update in parallel, one container at a time, and Dozzle always updates itself last.

To opt a container into scheduled updates, or to silence the check on one you pinned on purpose, add a label:

```yaml
    labels:
      - dev.dozzle.auto-update=true   # update on Dozzle's schedule
      - dev.dozzle.update-check=false # never check this one
```

> Only auto-update containers you are happy to see replaced without watching. A database on a floating tag like `postgres:latest` can jump to a new major version its data files cannot read. Updates also recreate the container, so anything in an anonymous volume is lost. Bind mounts like `/mnt/tank/configs/...` are safe.
{.is-warning}

To turn the registry checks off entirely, set `DOZZLE_IMAGE_CHECK_MODE=off` (or `manual` to check only when you ask).

## 3.4 App Icons and Container Links

Well-known images now show their app icon automatically. Add a `dev.dozzle.url` label and Dozzle puts a link to the app's web UI next to its name, so you can jump from the logs straight to the app:

```yaml
    labels:
      - dev.dozzle.url=https://jellyfin.yourdomain.com
      - dev.dozzle.name=Jellyfin
      - dev.dozzle.group=Media
```

If you use Traefik, Dozzle reads your router labels and prefills a suggested link for you. The URL must be a full `http` or `https` address.

## 3.5 Host Metrics

The host card can now show uptime, load average and disk usage. Load and uptime need the `/proc:/host/proc:ro` mount from section 1. Disk works out of the box.

To watch your pools too, mount each one under `/host/disks/<name>`. Dozzle only needs a mount point on the right filesystem, so point it at an empty folder rather than the whole pool:

```bash
mkdir -p /mnt/tank/.dozzle /mnt/bigdeal/.dozzle
```

```yaml
    volumes:
      - /mnt/tank/.dozzle:/host/disks/tank:ro
      - /mnt/bigdeal/.dozzle:/host/disks/bigdeal:ro
```

The bar shows whichever drive is fullest, and hovering it lists them all.

## 3.6 MCP Server for AI Assistants

Dozzle can expose a read-only MCP endpoint so Claude Desktop, Claude Code, VS Code and other assistants can list containers, read and search logs, and pull stats. Turn it on with:

```yaml
    environment:
      - DOZZLE_ENABLE_MCP=true
```

Then point your client at `http://your-server-ip:8888/api/mcp`. With login enabled, the client opens a browser tab the first time it connects, you sign in, approve it, and it handles its own tokens from there.

> With no login configured, the MCP endpoint is open to anyone who can reach Dozzle. Turn on login before enabling it.
{.is-warning}

# 4 · Monitoring Other Hosts

Run a Dozzle agent on any other Docker machine (a VPS, a Proxmox VM, a Pi) and add it from **Add host** at the bottom of the host list, or in the wizard.

```yaml
services:
  dozzle-agent:
    image: amir20/dozzle:latest
    command: agent
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    ports:
      - 7007:7007
```

1. Check the socket's group on that machine with `stat -c '%g' /var/run/docker.sock` and use that number under `group_add`. It is often different from TrueNAS.
2. In Dozzle, click **Add host** and enter the agent's address, for example `10.99.0.50:7007`.
3. The host appears in the sidebar right away, with its own uptime, load and disk on its card.

> Keep the agent's port `7007` off the public internet. Reach it over your LAN or a mesh VPN like NetBird.
{.is-info}

# <img src="/youtube.png" class="tab-icon"> 5 · Video

