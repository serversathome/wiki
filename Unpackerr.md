---
title: Unpackerr
description: A guide to installing Unpackerr in TrueNAS Scale as well as docker via compose
published: true
date: 2026-06-17T13:27:01.319Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:02:58.731Z
---

# <img src="/unpackerr.png" class="tab-icon"> What is Unpackerr?

**Unpackerr** runs as a daemon on your download host or seedbox. It checks for completed downloads and extracts them so Lidarr, Radarr, Readarr, and Sonarr may import them. If your problem is RAR files getting stuck in your activity queue, then this is your solution.

Not a Starr app user, and just need to extract files? It does that too. Unpackerr can run standalone and extract files found in a "watch" folder — point it at any download folder and it will happily extract everything that lands there.

Unpackerr extracts `rar`, `tar`, `tgz`, `gz`, `zip`, `7z`, `bz2`, `tbz2`, and `iso`, recursively, including archives nested inside archives. Multi-file and password-protected RAR and 7-Zip archives are supported.

> Unpackerr has **no web interface**. It runs silently in the background, so the only way to confirm it's working is by reading its logs. That's by design — once it's set up, you shouldn't have to look at it.
{.is-info}

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
| **Type** | Sonarr, Radarr, Lidarr, Readarr, or Whisparr |
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

> The wizard often pre-fills **two** Path boxes and **two** Protocol boxes. You only need one of each — remove the extras with the **✕**. Add a second only if you genuinely have a second value (e.g. a `Usenet` protocol alongside `Torrent`).
{.is-warning}

5. **User and Group Configuration** — leave at `568:568` (the host `apps` user). Make sure `568` has read/write on your downloads dataset.
6. **Storage Configuration** — click **Add**, set **Type** to **Host Path** (not ixVolume). Point the **Host Path** at the dataset where your media lives (ex. `/mnt/tank/media`) and set the **Mount Path** to the container side (ex. `/media`).
7. **Network Configuration** *(optional)* — enable the web server / metrics option to expose Prometheus metrics on port `5656` for the [Unpackerr Grafana dashboard](https://grafana.com/grafana/dashboards/18817-unpackerr/).
8. Scroll down and click **Install**. Once the app is **Running**, open its **Logs** to confirm it connected to each Starr app and is polling.

> **This is the step everyone gets wrong.** The **Mount Path** in Storage (plus its subfolder) must equal the **Path** you typed in Settings — and both must point to the *same place your Sonarr and qBittorrent already use*. If you mount `/mnt/tank/media → /media`, then the Path is `/media/downloads`. If these don't line up, Unpackerr starts perfectly and silently finds nothing to extract.
{.is-danger}

## <img src="/docker.png" class="tab-icon"> Docker Compose

Prefer to run it as code in Dockge? Use the official `golift/unpackerr` image. Edit the **URL**, **API Key**, and **PATHS_0** for each app you use, and delete the blocks for apps you don't run.

```yaml
services:
  unpackerr:
    image: golift/unpackerr
    container_name: unpackerr
    volumes:
      # You need at least this one volume mapped so Unpackerr can find your files to extract.
      # Make sure this matches your Starr apps; the folder mount (/downloads or /media) should be identical.
      - /mnt/tank/media:/media
    restart: unless-stopped
    user: 568:568
    environment:
      - TZ=America/New_York
      # General config
      - UN_DEBUG=false
      - UN_INTERVAL=2m
      - UN_START_DELAY=1m
      - UN_RETRY_DELAY=5m
      - UN_MAX_RETRIES=3
      - UN_PARALLEL=1
      - UN_FILE_MODE=0644
      - UN_DIR_MODE=0755
      # Sonarr Config
      - UN_SONARR_0_URL=http://sonarr:8989
      - UN_SONARR_0_API_KEY=YOUR_SONARR_API_KEY
      - UN_SONARR_0_PATHS_0=/media/downloads
      - UN_SONARR_0_PROTOCOLS=torrent
      - UN_SONARR_0_TIMEOUT=10s
      - UN_SONARR_0_DELETE_ORIG=false
      - UN_SONARR_0_DELETE_DELAY=5m
      # Radarr Config
      - UN_RADARR_0_URL=http://radarr:7878
      - UN_RADARR_0_API_KEY=YOUR_RADARR_API_KEY
      - UN_RADARR_0_PATHS_0=/media/downloads
      - UN_RADARR_0_PROTOCOLS=torrent
      - UN_RADARR_0_TIMEOUT=10s
      - UN_RADARR_0_DELETE_ORIG=false
      - UN_RADARR_0_DELETE_DELAY=5m
      # Lidarr Config
      - UN_LIDARR_0_URL=http://lidarr:8686
      - UN_LIDARR_0_API_KEY=YOUR_LIDARR_API_KEY
      - UN_LIDARR_0_PATHS_0=/media/downloads
      - UN_LIDARR_0_PROTOCOLS=torrent
      - UN_LIDARR_0_TIMEOUT=10s
      - UN_LIDARR_0_DELETE_ORIG=false
      - UN_LIDARR_0_DELETE_DELAY=5m
      # Readarr Config
      - UN_READARR_0_URL=http://readarr:8787
      - UN_READARR_0_API_KEY=YOUR_READARR_API_KEY
      - UN_READARR_0_PATHS_0=/media/downloads
      - UN_READARR_0_PROTOCOLS=torrent
      - UN_READARR_0_TIMEOUT=10s
      - UN_READARR_0_DELETE_ORIG=false
      - UN_READARR_0_DELETE_DELAY=5m
      # Web Server / Metrics (optional — for Grafana)
      - UN_WEBSERVER_METRICS=false
      - UN_WEBSERVER_LISTEN_ADDR=0.0.0.0:5656
```

> Unlike the TrueNAS app, the compose method **can** use container names like `http://sonarr:8989` for the URL — as long as Unpackerr and your Starr apps share the same Docker network.
{.is-info}

### Permissions & Folder Structure

- **PUID / PGID** — use a user/group with permission to your media folders. TrueNAS SCALE defaults to `568:568` for apps.
- 📌 See the [Folder-Structure](https://wiki.serversatho.me/Folder-Structure) guide for the recommended single-mount layout.

> For a full reference of every configuration option with examples, see the [Unpackerr documentation](https://unpackerr.zip/docs/install/configuration).
{.is-success}

# 2 · Configuration Notes

A few options worth knowing about, set as environment variables (Docker) or wizard fields (TrueNAS):

| Option | Default | What it does |
|--------|---------|--------------|
| `UN_INTERVAL` | `2m` | How often Unpackerr polls your Starr apps |
| `..._DELETE_DELAY` | `5m` | How long after import before extracted files are removed |
| `UN_PARALLEL` | `1` | How many extractions run at once (raise on powerful hardware) |
| `UN_MAX_RETRIES` | `3` | Retry attempts before giving up on an archive |
| `UN_FOLDER_0_PATH` | *(empty)* | Watch-folder path for non-Starr extraction |
| `UN_WEBSERVER_METRICS` | `false` | Exposes a Prometheus metrics endpoint on `:5656` |


> **Password-protected archives:** Unpackerr can crack password-protected RAR/7-Zip if given a password list, but a wrong password forces it to read the entire archive before failing. A long password list will dramatically slow extractions and hammer your disk — use sparingly.
{.is-warning}

# <img src="/patreon-light.png" class="tab-icon"> 3 · Video
[![](/unpackerr-patreon.png)](https://www.patreon.com/serversathome/posts/stop-rar-files-161337330)