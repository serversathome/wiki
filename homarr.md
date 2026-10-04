---
title: Homarr
description: A guide to deploying Homarr
published: true
date: 2026-10-04T19:30:28.147Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:05:10.051Z
---

# <img src="/homarr.png" class="tab-icon"> What is Homarr?

**Homarr** is a modern, sleek dashboard for your homelab that puts all of your apps and services at your fingertips. It uses drag-and-drop configuration with no YAML required, making it accessible for beginners while remaining powerful enough for advanced users. Version 2 is the biggest update in the project's history, adding community-built Custom Widgets, a rebuilt board editor and a built-in Assistant.

Key features include:

- Rebuilt drag-and-drop editor with Containers, fixed sidebars and separate desktop and mobile layouts
- 80+ built-in integrations (Sonarr, Radarr, Plex, Jellyfin, qBittorrent, Home Assistant and more)
- 20,000+ built-in icons
- Custom Widgets and the community Workshop for sharing widgets and Custom CSS
- Homarr Assistant and an MCP endpoint for AI tools, with permission checks on every action
- Built-in authentication and authorization (credentials, OIDC, LDAP)
- Docker and Podman discovery for adding apps and integrations automatically

<div class="glance">
  <div><span>Port</span><b><code>7575</code></b></div>
  <div><span>Deploy via</span><b>TrueNAS app or Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/homarr-labs/homarr"><i class="mdi mdi-github"></i>Project</a>
    <a href="https://homarr.dev/docs/"><i class="mdi mdi-book-open-variant"></i>Docs</a>
    <a href="https://apps.truenas.com/catalog/homarr_community/"><i class="mdi mdi-server"></i>TrueNAS app</a>
  </div>
</div>

# 1 · Deploy Homarr
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

```yaml
services:
  homarr:
    image: ghcr.io/homarr-labs/homarr:latest
    container_name: homarr
    restart: unless-stopped
    ports:
      - "7575:7575"
    environment:
      - SECRET_ENCRYPTION_KEY=your_64_character_hex_string
      - PUID=568
      - PGID=568
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /mnt/tank/configs/homarr/appdata:/appdata
```

1. Generate a secret encryption key using `openssl rand -hex 32`
2. Replace `your_64_character_hex_string` with your generated key
3. Create the config folder and hand it to the apps user before the first start:

```bash
mkdir -p /mnt/tank/configs/homarr/appdata
chown -R 568:568 /mnt/tank/configs/homarr
```

> Keep a copy of your `SECRET_ENCRYPTION_KEY` somewhere safe. Homarr uses it to encrypt every API key and password stored in the database, and you need the same key for restores and upgrades. If the key is missing at startup, the container exits and prints a random key in the error message.
{.is-info}

> The Docker socket mount is optional and only needed for Docker discovery and container management. Because the container runs as `568:568`, it also needs the socket's group. Find it with `stat -c '%g' /var/run/docker.sock`, set `PGID` to that number and run `chown -R 568:<that-gid> /mnt/tank/configs/homarr`. If you don't need Docker features, remove the `/var/run/docker.sock` line and leave `PGID=568`.
{.is-info}

## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Navigate to **Apps** in the TrueNAS UI
2. Search for "**Homarr**" (Community train)
3. Click **Install**
4. Configure the following settings:
   - **Secret Encryption Key**: Generate with `openssl rand -hex 32`
   - **Mount Docker Socket**: Enable if you want Docker integration
   - **User ID** / **Group ID**: `568` (default)
   - **WebUI Port**: `7575` (default)
   - **Homarr Data Storage**: Type **Host Path**, path `/mnt/tank/configs/homarr`
5. Click **Install**

> As of early October 2026 the TrueNAS catalog still ships Homarr v1.77.2. When the catalog moves to v2, updating the app from the **Installed** screen runs the database migration automatically. If you want v2 right now, deploy it with the Docker compose above using **Install via YAML**.
{.is-info}

# 2 · Configuration

## 2.1 Onboarding

Browse to `http://<server-ip>:7575`. A fresh install redirects to the setup studio, where you can choose **Get started** or **Restore backup** (for a compatible SQLite backup). Setup is split into six stages:

1. **Essentials** - Language, theme, your usual server address and the analytics preference
2. **Discover** - Shows which capabilities Homarr can reach (database, Docker, Kubernetes, Assistant, Workshop)
3. **Connect** - Import services Homarr detected or add integrations by hand. Anything unfinished can be skipped
4. **Board** - Board name, colors, corner radius, column count and optional sidebars
5. **Extend** - Optionally connect Workshop, set up an Assistant provider or review the MCP endpoint
6. **Review** - Check the summary, then build the board

Integrations, Workshop, Assistant and MCP are all optional. Skip them if you just want a working board and come back later.

> If onboarding shows up again after a restart, your `/appdata` folder isn't persisting or isn't writable. Check the volume path and the `568:568` ownership.
{.is-warning}

## 2.2 Core Concepts

| Term | What it is |
|------|------------|
| Board | A dashboard page holding apps, widgets and Containers. Can be public or restricted to users and groups |
| App | A saved shortcut: name, URL, icon and optional status check |
| Integration | A server-side connection from Homarr to a service such as Sonarr or Home Assistant |
| Widget | A board tile. Some work alone, others need one or more integrations |
| Container | A group of apps, widgets and other Containers. Replaces v1 Groups and sections |
| Rail | A fixed sidebar that stays in place while the board scrolls |


## 2.3 Building Your Board

1. Enter edit mode from the board header
2. Open the add-content menu to create an **App**, **Integration**, **Widget** or **Container** without leaving the board
3. When adding apps, click one to add it immediately, or choose **Select multiple** to add several at once
4. Drag and resize items. Homarr previews the move and only saves valid layouts, so a bad drop rolls back instead of scrambling the board
5. Save the board to keep the layout

A few things worth knowing in the new editor:

- Hold <kbd>Ctrl</kbd> (or <kbd>Cmd</kbd> on macOS) and click to select several items, then use **Move to** to drop them into a Container
- Containers can be collapsed and moved with everything inside them
- Every board has a **Base** layout and a **Mobile** layout. **Reset from Base** builds a mobile starting point from your desktop layout
- Turn on a left or right sidebar and set its width under board settings to keep shortcuts on screen while the board scrolls

## 2.4 Integrations

In v2, integrations live in their own place instead of inside each app tile. Go to **Management → Integrations**, pick a service, enter its base URL and credentials, then test the connection. Homarr won't save a new integration until the test passes. Once saved, you can link an integration to an app and select it in any widget that supports it.

Homarr supports 80+ services, including:

- **Media Management**: Sonarr, Radarr, Lidarr, Readarr, Prowlarr, Jackett, Autobrr
- **Media Servers and Requests**: Plex, Jellyfin, Jellystat, Overseerr, Jellyseerr, Komga
- **Download Clients**: qBittorrent, Transmission, Deluge, SABnzbd, NZBGet
- **DNS/Ad-blocking**: Pi-hole, AdGuard Home
- **Monitoring and Networking**: Netdata, Prometheus, Scrutiny, Gatus, Healthchecks, Caddy, Frigate
- **Home and Collections**: Home Assistant, Mealie, Tandoor, Karakeep, Linkwarden, Homebox
- **Other**: TrueNAS, Docker, Podman

Integration permissions come in three levels:

| Level | Allows |
|-------|--------|
| Use | Select the integration and read its data |
| Interact | Run supported actions, such as pausing a download |
| Full | Change the integration's settings and call any API endpoint |


> Integration URLs must be reachable from inside the Homarr container, so use your server's IP address (for example `http://192.168.1.100:8989`) rather than Docker container names unless Homarr shares a network with that app. The browser-facing URL you click on can be different; set that on the linked app.
{.is-warning}

## 2.5 Widgets

| Widget | Description |
|--------|-------------|
| Weather | Current conditions and forecast (redesigned in v2) |
| Clock | Clock with timezone support (redesigned in v2) |
| Calendar | iCal feeds plus Sonarr/Radarr release calendars |
| Statistics | Combines metrics from several integrations in one grid of cards, rows or tables (new in v2) |
| Air Quality | Current AQI and trend for a location (new in v2) |
| Countdown | Time remaining until an event (new in v2) |
| Timer | Start, pause and reset a timer from the board (new in v2) |
| Docker | Container status and management |
| Iframe | Embed external pages |
| RSS | RSS feed reader |


Many widgets now have an **advanced view**. Hover over the widget and hold <kbd>Shift</kbd> for half a second to see more data and controls.

## 2.6 Custom Widgets and Workshop

Custom Widgets let you build a widget for almost any service, even one Homarr doesn't support natively. Write one yourself or describe what you want to the Assistant, then test it in the workbench. Custom Widgets can use a saved integration as their data source, so you never paste API keys into widget code.

**Workshop** is the community library at [homarr.dev/workshop](https://homarr.dev/workshop). Browse, install, vote on and update widgets and Custom CSS that other users have shared. Your URLs and credentials stay on your own instance.

## 2.7 Assistant and MCP

Press <kbd>Shift</kbd> + <kbd>A</kbd> to open the Homarr Assistant. It can answer questions about your boards, apps, integrations and Docker hosts, make changes and build Custom Widgets. It follows your user permissions: reads run automatically, but changes wait for your approval.

External AI clients can use the same tools through the MCP endpoint at `/api/mcp`, authenticated with a Homarr API key or OAuth.

> If Homarr sits behind a reverse proxy that doesn't pass the original host and protocol, add `BASE_URL=https://homarr.yourdomain.com` to the environment so MCP OAuth redirects use your public address.
{.is-info}

## 2.8 Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| <kbd>Ctrl</kbd>/<kbd>Cmd</kbd> + <kbd>K</kbd> | Command menu: find pages, settings, users and commands |
| <kbd>Shift</kbd> + <kbd>C</kbd> | Board switcher |
| <kbd>Shift</kbd> + <kbd>A</kbd> | Open the Assistant |
| Hold <kbd>Shift</kbd> over a widget | Advanced widget view |
| <kbd>Esc</kbd> while dragging | Put the item back where it was |

