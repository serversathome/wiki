---
title: Maintainerr
description: A guide to deploying Maintainer via docker
published: true
date: 2026-09-08T13:07:59.352Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:06:10.333Z
---

# <img src="/maintainerr.png" class="tab-icon"> What is Maintainerr?

**Maintainerr** is the janitor your media server never had. It builds rules from data across Plex, Jellyfin, Emby, Radarr, Sonarr, Seerr, Tautulli and Streamystats, gathers the matching titles into a collection, holds them for a grace period so your users can catch up, and then cleans them off the server, the *arrs and Seerr automatically.

Think of it as the opposite of Seerr. Seerr adds media to your library; Maintainerr takes it back out once nobody is watching it. Set the rules once and it runs on its own schedule from there.

Typical uses:

- Delete movies nobody has watched in 12 months
- Pin a "Leaving Soon" shelf to the Plex home screen before anything gets removed
- Unmonitor or delete in Radarr/Sonarr and clear the matching Seerr request in one action
- Remove the completed download from qBittorrent once seeding is finished

<div class="glance">
  <div><span>Port</span><b><code>6246</code></b></div>
  <div><span>Deploy via</span><b>TrueNAS app or Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty intermediate">Intermediate</b></div>
  <div class="glance-links">
    <a href="https://youtu.be/u8k-IlkShKs"><i class="mdi mdi-youtube"></i>Video walkthrough</a>
  </div>
</div>



# 1 · Deploy Maintainerr

# {.tabset}

## <img src="/docker.png" class="tab-icon"> Docker

```yaml
services:
  maintainerr:
    image: ghcr.io/maintainerr/maintainerr:latest
    container_name: maintainerr
    user: "568:568"
    environment:
      - TZ=America/New_York
    restart: unless-stopped
    ports:
      - "6246:6246"
    volumes:
      - /mnt/tank/configs/maintainerr:/opt/data
```


Optional environment variables:

| Variable | Purpose |
|----------|---------|
| `BASE_PATH` | Serve Maintainerr from a subdirectory, e.g. `/maintainerr` |
| `UI_PORT` | Change the UI port from the default `6246` |
| `UI_HOSTNAME` | Set to `::` to listen on IPv6 instead of `0.0.0.0` |
| `GITHUB_TOKEN` | Raises the GitHub API rate limit from 60/hr to 5000/hr |
{.dense}

> Overlays and collection posters use `sharp`, which needs a CPU supporting `x86-64-v2`. Maintainerr still starts on older CPUs but those image features stay disabled. If you run it in a Proxmox VM, set the CPU type to `host` — the default `kvm64` does not expose `x86-64-v2`. This does not affect arm64.
{.is-warning}

## <img src="/truenas.png" class="tab-icon"> TrueNAS

Maintainerr is in the **Community** train of the TrueNAS Apps catalog.

1. Navigate to **Apps** in the TrueNAS UI and click **Discover Apps**
2. Search for "Maintainerr" and click **Install**
3. Configure the following settings:
   - **Timezone**: your local timezone (defaults to `Etc/UTC`)
   - **User ID / Group ID**: leave at `568` / `568` unless you have a reason to change them
   - **WebUI Port**: `30180` by default — change it to `6246` if you want it to match the Docker install
   - **Maintainerr Data Storage**: choose `Host Path` and point it at `/mnt/tank/configs/maintainerr`, or leave it on `ixVolume` to let TrueNAS create the dataset for you
   - **Resources**: 2 CPUs and 4096 MB memory by default, which is plenty
4. Click **Install**
5. Once the app shows **Running**, click **Web Portal**

> The TrueNAS app runs as UID/GID `568` (the `apps` user). If you pick a host path, make sure that dataset is owned by `apps:apps` or the container cannot create its database.
{.is-info}

> Community train apps are packaged and maintained by the TrueNAS community, not by the Maintainerr developers. The app version usually trails the upstream release slightly. If you want same-day updates, run the Docker version instead.
{.is-info}

# 2 · Configuration

All configuration lives inside the app itself — there are no config files to edit. On first launch you should land on the settings page automatically. If you don't, refresh the page.

> Every service URL field expects a full base URL starting with `http://` or `https://`. Trailing slashes are stripped for you on save.
{.is-info}

## 2.1 General

| Setting | Description |
|---------|-------------|
| Hostname | The hostname or IP of the host running Maintainerr |
| API key | Maintainerr's own API key — reserved for future use |
| Display language | Language for the web UI |


## 2.2 Media Server

Pick **one** media server. Plex, Jellyfin and Emby cannot run side by side in a single instance — if you need both, deploy a second Maintainerr container with its own data volume.

### Plex

Authenticate with a Plex **admin** account and Maintainerr discovers your servers automatically. Pick the one you want from the dropdown.

> If you're running a local Plex instance, set Plex's **Secure connections** network setting to `Preferred` instead of `Required`, or the connection test will fail.
{.is-warning}

If the server dropdown comes back empty, Plex is publishing only a local connection that Maintainerr can't reach. Fix it on the Plex side:

1. In Plex Web, go to **Settings → (your server) → Network** and click **Show Advanced**
2. Add a reachable address to **Custom server access URLs** — your LAN address such as `http://192.168.1.10:32400` works fine for a homelab
3. Save, restart Plex, then hit the **Refresh** icon next to the server selector in Maintainerr

> If Plex and Maintainerr share a Docker network, skip discovery entirely: open **Advanced Settings**, enable **Manual connection override**, and point it at `http://plex:32400`. Note that manual mode disables automatic reconnection.
{.is-info}

### Jellyfin

| Setting | Description |
|---------|-------------|
| Jellyfin URL | Domain or local IP of the Jellyfin host |
| API key | Generated in Jellyfin under Dashboard → API Keys |
| Admin User | Click **Test Connection** first to populate the list |


### Emby

| Setting | Description |
|---------|-------------|
| Emby URL | Domain or local IP of the Emby host |
| API key | From `Dashboard → Advanced → API Keys`, or use **Sign in with Emby** |
| Admin User | Click **Test Connection** to load available admin users |


> Emby Connect is not supported. Maintainerr authenticates directly against the server.
{.is-info}

## 2.3 Radarr, Sonarr and Seerr

These are optional but you'll want them. Without Radarr/Sonarr, Maintainerr can only remove items from the media server itself — it can't unmonitor, delete files, or clear requests.

| Setting | Description |
|---------|-------------|
| Server Name | Friendly label to identify the server |
| Hostname or IP | Domain or local IP of the host |
| Port | The port Radarr/Sonarr runs on |
| Base URL | The URL base if you set one |
| API key | From the app's Settings → General page |


> Enter the Base URL **without** a leading slash — `radarr`, not `/radarr`.
{.is-warning}

Seerr is configured with just a URL and API key. Add it if you want Seerr rule parameters, want requests cleared when media is deleted, or want to see who requested something in pre-deletion notifications.

## 2.4 Download Client

The **Settings → Download Client** page only appears once Radarr or Sonarr is configured. qBittorrent is the only supported client (4.3.4 or newer, Web UI enabled).

| Setting | Description |
|---------|-------------|
| URL | Base URL of the qBittorrent WebUI |
| Username / Password | Leave blank if auth is bypassed for your subnet |
| Delete downloaded data | Also removes files from disk — turn **off** if you cross-seed |
| Fallback seeding ratio | Applies only to downloads qBittorrent isn't limiting (min 0.5) |



## 2.5 Optional Integrations

| Service | Notes |
|---------|-------|
| Tautulli | Plex only — enables watch-history rule parameters |
| Streamystats | Jellyfin only — the settings page appears once Jellyfin is active |
| Tracearr | Watch history rules; only tracks events recorded after it was connected |
| Metadata | Optional TMDB key for your own quota, plus a TVDB key as a second source |


# 3 · Building Your First Rule

The workflow is always the same: a rule matches media, matched media lands in a collection, the collection holds it for X days, then the collection's action fires.

1. Go to **Rules** and click the **+** button
2. Give it a name and pick the library it applies to
3. Set the **Collection handling** — how many days items are held before the action runs
4. Choose the **Action**: delete, unmonitor and delete files, unmonitor and keep files, or change quality profile
5. Add your rule conditions, combining them with AND/OR logic
6. Save, then use **Run rules** to test it before letting the schedule take over

A common starter rule for movies:

> **Delete unwatched movies after a year**
> - [x] Plex → date added → is older than → 365 days
> - [x] Plex → view count → equals → 0
> - [x] Seerr → requested by → not in → (your own username)

> Test every new rule with a manual run and look at what lands in the collection **before** you shorten the grace period. Maintainerr deletes files for real — there is no undo.
{.is-danger}

Use the collection's grace period as your safety net. Setting it to 30 days and pinning the collection to the Plex home screen as a "Leaving Soon" shelf gives your users a fair warning and gives you a month to notice a rule that's matching too much.


# <img src="/youtube.png" class="tab-icon"> 4 · Video
https://youtu.be/u8k-IlkShKs