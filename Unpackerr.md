---
title: Unpackerr
description: A guide to installing Unpackerr in TrueNAS Scale as well as docker via compose
published: true
date: 2026-10-26T11:32:43.777Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:02:58.731Z
---

# <img src="/unpackerr.png" class="tab-icon"> What is Unpackerr?

**Unpackerr** runs as a daemon on your download host or seedbox. It checks for completed downloads and extracts them so Lidarr, Radarr, Readarr, and Sonarr may import them. If your problem is RAR files getting stuck in your activity queue, then this is your solution.

Not a Starr app user, and just need to extract files? It does that too. Unpackerr can run standalone and extract files found in a "watch" folder — point it at any download folder and it will happily extract everything that lands there.

Unpackerr extracts `rar`, `tar`, `tgz`, `gz`, `zip`, `7z`, `bz2`, `tbz2`, and `iso`, recursively, including archives nested inside archives. Multi-file and password-protected RAR and 7-Zip archives are supported.

> **New in 1.0: a web interface.** Unpackerr used to have no UI at all. Since v1.0.0 it serves a dashboard on port `5656` with a live queue (retry and forget), history, logs and a **Settings** page where you can add your Starr apps and watch folders and **Test** them before saving. Upgrading from 0.x? See [3 · Upgrading from 0.x](#h-3-upgrading-from-0x).
{.is-info}

<div class="glance">
  <div><span>Deploy via</span><b>TrueNAS app or Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://unpackerr.zip/docs/install/configuration"><i class="mdi mdi-book-open-variant"></i>Docs</a>
    <a href="https://github.com/Unpackerr/unpackerr/releases/tag/v1.0.0"><i class="mdi mdi-github"></i>v1.0 Release</a>
  </div>
</div>

# 1 · Deploy Unpackerr
# {.tabset}
## <img src="/truenas.png" class="tab-icon"> TrueNAS

There is an official Unpackerr app in the TrueNAS **Community** catalog, so no Dockge or compose file is required.

1. Go to **Apps → Discover Apps**, search for **Unpackerr**, and click **Install**.
2. **Application Name** — leave as `unpackerr`.
3. **Unpackerr Configuration → Timezone** — set your timezone so log timestamps are readable.
4. **Unpackerr Configuration → Settings** — click **Add** for each Starr app you want to link, and fill in the fields below.

### Settings (per Starr app)

| Field | What to enter |
|-------|---------------|
| **Type** | Sonarr, Radarr, Lidarr, Readarr, or Whisparr (see note below) |
| **URL** | Your TrueNAS LAN IP + the app's port (see warning below) |
| **API Key** | From the app: **Settings → General → Security → API Key** |
| **Path** | The container path Unpackerr should check — must match your Storage mount (ex. `/media/downloads`) |
| **Protocols** | `Torrent`, `Usenet`, or both — match your download clients |
| **Timeout** | `10` seconds (default) |
| **Delete Delay** | `300` seconds / 5 min (default) |
| **Delete Original** | Leave **unchecked** if you seed |
| **Syncthing** | Leave off unless the path is Syncthing-managed |


> **The URL must use your host IP, not a container name.** Inside a compose file `http://sonarr:8989` works because the apps share a Docker network. TrueNAS catalog apps **do not** share a network by default — they publish ports on the host — so use your TrueNAS box's LAN IP and the app's port, e.g. `http://192.168.1.50:8989` (Radarr `7878`, Lidarr `8686`, Readarr `8787`, Whisparr `6969`).
{.is-danger}

> **Whisparr** is no longer a separate app type in Unpackerr 1.0; it is set up as a Radarr instance. If the TrueNAS app still offers a Whisparr type, add Whisparr in the Unpackerr web UI instead.
{.is-info}

> The wizard often pre-fills **two** Path boxes and **two** Protocol boxes. You only need one of each — remove the extras with the **✕**. Add a second only if you genuinely have a second value (e.g. a `Usenet` protocol alongside `Torrent`).
{.is-warning}

5. **User and Group Configuration** — leave at `568:568` (the host `apps` user). Make sure `568` has read/write on your downloads dataset.
6. **Storage Configuration** — click **Add**, set **Type** to **Host Path** (not ixVolume). Point the **Host Path** at the dataset where your media lives (ex. `/mnt/tank/media`) and set the **Mount Path** to the container side (ex. `/media`).
    - Click **Add** again for the config: **Host Path** `/mnt/tank/configs/unpackerr` (create the dataset first, owned by `568`), **Mount Path** `/config`. Unpackerr 1.0 writes its settings to `/config/unpackerr.conf` when you save in the web UI; without this mount, those changes are lost on restart.
7. **Network Configuration** — make sure the web UI port `5656` is published so you can reach the dashboard.
8. Scroll down and click **Install**. Once the app is **Running**, open its **Logs**: confirm it connected to each Starr app, and copy the **temporary UI password** (see [Logging in to the Web UI](#h-logging-in-to-the-web-ui)).

> **This is the step everyone gets wrong.** The **Mount Path** in Storage (plus its subfolder) must equal the **Path** you typed in Settings — and both must point to the *same place your Sonarr and qBittorrent already use*. If you mount `/mnt/tank/media → /media`, then the Path is `/media/downloads`. If these don't line up, Unpackerr starts perfectly and silently finds nothing to extract.
{.is-danger}

## <img src="/docker.png" class="tab-icon"> Docker Compose

Prefer to run it as code in Dockge? Use the official `golift/unpackerr` image. Since 1.0 the compose file only needs the mounts, the port and the user: your Starr apps and every other setting are added in the web UI. Create a `configs/unpackerr` dataset for the `/config` mount first.

```yaml
services:
  unpackerr:
    image: golift/unpackerr
    container_name: unpackerr
    restart: unless-stopped
    user: 568:568
    ports:
      - 5656:5656
    environment:
      - TZ=America/New_York
    volumes:
      # Must match your Starr apps; the folder mount (/media) should be identical.
      - /mnt/tank/media:/media
      - /mnt/tank/configs/unpackerr:/config
```

On first start Unpackerr creates `/config/unpackerr.conf` with the defaults. Then [log in to the web UI](#h-logging-in-to-the-web-ui) and open **Settings**:

1. Add **Sonarr** and **Radarr** (and Lidarr / Readarr if you run them) with the **URL**, **API Key** (from the app: **Settings → General → Security → API Key**), **Path** `/media/downloads` and **Protocols** `torrent`.
1. Click **Test** on each row to check it can reach the app, then **Save**.
1. Optional: add a **watch folder** for non-Starr extraction (set its interval to `1s`, see [Configuration Notes](#h-2-configuration-notes)).

> The compose method **can** use container names like `http://sonarr:8989` for the URL — as long as Unpackerr and your Starr apps share the same Docker network. Otherwise use your host IP and the app's port.
{.is-info}

### Optional: Configure with Environment Variables

You can still set everything in the compose file instead, for example to keep the setup as code. Add these under `environment:`, one block per app:

```yaml
      # Sonarr
      - UN_SONARR_0_URL=http://sonarr:8989
      - UN_SONARR_0_API_KEY=YOUR_SONARR_API_KEY
      - UN_SONARR_0_PATHS_0=/media/downloads
      - UN_SONARR_0_PROTOCOLS=torrent
      # Radarr
      - UN_RADARR_0_URL=http://radarr:7878
      - UN_RADARR_0_API_KEY=YOUR_RADARR_API_KEY
      - UN_RADARR_0_PATHS_0=/media/downloads
      - UN_RADARR_0_PROTOCOLS=torrent
```

Lidarr and Readarr follow the same pattern (`UN_LIDARR_0_*`, `UN_READARR_0_*`). Every option and its variable name is in the [official compose example](https://github.com/Unpackerr/unpackerr/blob/main/examples/docker-compose.yml).

> **Environment variables lock settings in the UI.** Any field set by a `UN_*` variable is read-only in the web UI until you remove that variable, so pick one place to manage each setting.
{.is-warning}

### Permissions & Folder Structure

- **user** — use a user/group with permission to your media folders and the `configs/unpackerr` dataset. TrueNAS SCALE defaults to `568:568` for apps. The 1.0 image runs as the compose `user:`; it does not read `PUID` / `PGID`.
- 📌 See the [Folder-Structure](https://wiki.serversatho.me/Folder-Structure) guide for the recommended single-mount layout.

> For a full reference of every configuration option with examples, see the [Unpackerr documentation](https://unpackerr.zip/docs/install/configuration).
{.is-success}

### Logging in to the Web UI

Open `http://YOUR-IP:5656`. On first start Unpackerr generates a **temporary password** for the user `admin` and prints it **once** in the logs:

```
Generated temporary UI password for user admin: ...
```

Find it with `docker logs unpackerr` (or the app's **Logs** in TrueNAS), log in, and change it in the UI. Lost it? Add `command: --reset` to the compose service, start it once, copy the new password from the logs, then remove the line again.

# 2 · Configuration Notes

A few options worth knowing about. Set them in the web UI's **Settings**, or as environment variables (Docker) / wizard fields (TrueNAS):

| Option | Default | What it does |
|--------|---------|--------------|
| `UN_INTERVAL` | `2m` | How often Unpackerr polls your Starr apps |
| `..._DELETE_DELAY` | `5m` | How long after import before extracted files are removed |
| `UN_PARALLEL` | `1` | How many extractions run at once (raise on powerful hardware) |
| `UN_MAX_RETRIES` | `2` | Retry attempts before giving up on an archive |
| `UN_FOLDER_0_PATH` | *(empty)* | Watch-folder path for non-Starr extraction |
| `UN_FOLDER_0_INTERVAL` | `0s` (off) | Polling interval for that watch folder. **Set `1s` on Docker**, see below |
| `UN_WEBSERVER_LISTEN_ADDR` | `0.0.0.0:5656` | Address of the web UI and API. Empty turns the HTTP server off |
| `UN_WEBSERVER_METRICS` | `false` | Exposes a Prometheus metrics endpoint at `/metrics` on `:5656` |

> **Watch folders on Docker:** 1.0 no longer polls watch folders every second; each folder now has its own `interval`, which is off by default (file-system events only). Inside Docker, and on SMB/CIFS shares, those events often don't arrive, so set `UN_FOLDER_0_INTERVAL=1s` or new archives may never be noticed. The old global `UN_FOLDERS_INTERVAL` is ignored.
{.is-warning}

> **Metrics need an API key now.** `/metrics` only answers requests with an API key that has the `system:metrics:read` permission (as `Authorization: Bearer <key>` or an `X-Api-Key` header). Create one in the web UI and add it to your Prometheus scrape config for the [Unpackerr Grafana dashboard](https://grafana.com/grafana/dashboards/18817-unpackerr/).
{.is-info}

> **Webhooks:** 1.0 recognises Notifiarr, Discord, Telegram, Slack, Pushover, Gotify, ntfy, Apprise and Mattermost from the URL and formats the message for them. Discord and Telegram edit one message as the extraction progresses instead of posting a new one each time.
{.is-success}


> **Password-protected archives:** Unpackerr can crack password-protected RAR/7-Zip if given a password list, but a wrong password forces it to read the entire archive before failing. A long password list will dramatically slow extractions and hammer your disk — use sparingly.
{.is-warning}

# 3 · Upgrading from 0.x

Unpackerr 1.0 is a big release with a few breaking changes. If you already run Unpackerr:

1. **Add a `/config` mount** (see the compose file above). Without it, anything you save in the new web UI is lost when the container restarts.
1. **Publish port `5656`** and log in with the temporary password from the logs. The web UI turns on by itself, because the HTTP server now starts whenever `UN_WEBSERVER_LISTEN_ADDR` is set and it defaults to `0.0.0.0:5656`. To keep it off, set `UN_WEBSERVER_LISTEN_ADDR=` (empty).
1. **Switch `PUID` / `PGID` to `user: 568:568`** if your compose still uses them.
1. **Whisparr:** move any `UN_WHISPARR_*` variables to a Radarr instance (`UN_RADARR_whisparr_*`), or add Whisparr as a Radarr instance in the UI.
1. **Watch folders:** add `UN_FOLDER_0_INTERVAL=1s` for each watch folder (see above).
1. **Metrics:** if you scrape `/metrics`, create an API key with `system:metrics:read` and send it with the scrape.

> Archives may now unpack to up to **7.5×** their compressed size for Starr apps (was 5×) before Unpackerr treats them as suspicious, and nested archives no longer count toward that limit. Watch folders have no limit unless you set `max_ratio`.
{.is-info}

For every detail, see the [v1.0.0 release notes](https://github.com/Unpackerr/unpackerr/releases/tag/v1.0.0) and the [upgrade guide](https://unpackerr.zip/docs/install/upgrading).

# <img src="/patreon-light.png" class="tab-icon"> 4 · Video
[![](/unpackerr-patreon.png)](https://www.patreon.com/serversathome/posts/stop-rar-files-161337330)