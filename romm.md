---
title: RomM
description: A guide to deploying RomM on TrueNAS and Docker
published: true
date: 2026-09-28T10:36:49.460Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:08:07.432Z
---

# ![](/romm.png){class="tab-icon"} What is RomM?
RomM (ROM Manager) allows you to scan, enrich, browse and play your game collection with a clean and responsive interface. With support for multiple platforms, various naming schemes, and custom tags, RomM is a must-have for anyone who plays on emulators.

RomM 5.x rebuilt the interface from the ground up with a universal input model that works with mouse, touch, keyboard and gamepad, and added per-user and per-group permissions, shared saves and savestates, server-side ROM patching, and emulator streaming. RomM 5.3 added a fully custom library structure, physical games (add a cartridge or disc by name or barcode), walkthroughs, a Jukebox soundtrack player, js-dos and PICO-8 in the browser, and automatic save sync.

<div class="glance">
  <div><span>Port</span><b><code>30061</code></b></div>
  <div><span>Deploy via</span><b>TrueNAS app or Docker compose</b></div>
  <div><span>Containers</span><b>2 services</b></div>
  <div><span>Depends on</span><b>MariaDB (compose) or Postgres (TrueNAS)</b></div>
  <div><span>Difficulty</span><b class="difficulty intermediate">Intermediate</b></div>
  <div class="glance-links">
    <a href="https://github.com/rommapp/romm"><i class="mdi mdi-github"></i>Project</a>
    <a href="https://docs.romm.app"><i class="mdi mdi-book-open-variant"></i>Docs</a>
  </div>
</div>

# 1 · Deploy RomM
> **RomM 5.3 will not start without a declared library structure.** RomM no longer guesses your folder layout. Until `filesystem.structure` is set in `config.yml`, the container refuses to start. Both tabs below cover how to add it.
{.is-danger}

# {.tabset}
## <img src="/truenas.png" class="tab-icon"> TrueNAS
![screenshot_from_2025-02-24_06-38-49.png](/screenshot_from_2025-02-24_06-38-49.png)

RomM lives in the **Community** train of the TrueNAS Apps catalog.

1. Set a secure database password
1. Set a secure redis password
1. Generate an auth secret key using `openssl rand -hex 32` in the terminal
1. Input any client IDs or API keys you have (optional)
1. Set **Host Path** for the library storage, resources storage, config storage, assets storage, and Postgres Data Storage

> When setting the Host Path for Postgres Data Storage be sure to check the box for **Automatic Permissions**!
{.is-warning}
6. Leave the User and Group ID at `568` (the `apps` user), and make sure `568` has ACL access to every dataset you point the app at
6. Increase Resource limits (optional, but recommended on large libraries since 5.3 runs 4 scan workers and 4 API workers by default)
6. Before starting the app, create `config.yml` in the dataset you chose for **config storage** with the library structure from the Docker Compose tab, and `chown 568:568` it

> If the app goes into a crash loop after updating to 5.3, the missing `filesystem.structure` block is the first thing to check. The app logs print the exact template it expects.
{.is-warning}

> The catalog app provisions **PostgreSQL** and a separate Redis container, unlike the MariaDB layout in the Docker Compose tab. Both are fully supported by RomM, but you cannot switch between the two deployment methods without migrating the database.
{.is-info}

## <img src="/docker.png" class="tab-icon"> Docker Compose
```yaml
services:
    romm:
        image: rommapp/romm:latest
        container_name: romm
        restart: unless-stopped
        user: 568:568
        environment:
            - TZ=America/New_York
            - DB_HOST=romm-db
            - DB_NAME=romm
            - DB_USER=romm-user
            - DB_PASSWD=dbpassword
            - ROMM_AUTH_SECRET_KEY= # Generate a key with `openssl rand -hex 32`
            - ROMM_BASE_URL=http://your-server-ip:30061
            - HASHEOUS_API_ENABLED=true # Free, no account needed, gives you hash matching immediately
            - STEAM_API_ENABLED=true # Free, no key needed, metadata for PC platforms
            - IGDB_CLIENT_ID= # Generate an ID and SECRET in IGDB
            - IGDB_CLIENT_SECRET= # https://api-docs.igdb.com/#account-creation
            - SCREENSCRAPER_USER= # https://www.screenscraper.fr/
            - SCREENSCRAPER_PASSWORD=
            - STEAMGRIDDB_API_KEY= # https://www.steamgriddb.com/profile/preferences/api
            - RETROACHIEVEMENTS_API_KEY= # https://retroachievements.org/controlpanel.php
        volumes:
            - /mnt/tank/media/romm/library:/romm/library
            - /mnt/tank/media/romm/resources:/romm/resources
            - /mnt/tank/media/romm/assets:/romm/assets
            - /mnt/tank/configs/romm:/romm/config
            - /mnt/tank/configs/romm/redis-data:/redis-data
        ports:
            - 30061:8080
        depends_on:
            romm-db:
                condition: service_healthy
                restart: true

    romm-db:
        image: mariadb:11.4
        container_name: romm-db
        restart: unless-stopped
        environment:
            - TZ=America/New_York
            - MARIADB_ROOT_PASSWORD=supersecret
            - MARIADB_DATABASE=romm
            - MARIADB_USER=romm-user
            - MARIADB_PASSWORD=dbpassword
        volumes:
            - /mnt/tank/configs/romm/mysql_data:/var/lib/mysql
        healthcheck:
            test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
            start_period: 30s
            start_interval: 10s
            interval: 10s
            timeout: 5s
            retries: 5
```

Before deploying, create the host directories and hand them to the `apps` user:

```bash
mkdir -p /mnt/tank/media/romm/{library/roms,library/bios,resources,assets}
chown -R 568:568 /mnt/tank/media/romm
chmod 770 -R /mnt/tank/media/romm
```

Then create `config.yml` with your library structure. This layout matches the folders above (`library/roms/<platform>` and `library/bios/<platform>`):

```bash
mkdir -p /mnt/tank/configs/romm
cat > /mnt/tank/configs/romm/config.yml << 'EOF'
filesystem:
    structure:
        default: "roms/{platform}/{game}"
        firmware: "bios/{platform}"
EOF
chown 568:568 /mnt/tank/configs/romm /mnt/tank/configs/romm/config.yml
```

> If your platforms sit at the top of the library with `roms` inside each one (`library/<platform>/roms`), use `default: "{platform}/roms/{game}"` and `firmware: "{platform}/bios"` instead. See [Custom library structure](#custom-library-structure) below for per-platform layouts.
{.is-info}

> **Keep `library` and `resources` on the same ZFS dataset.** RomM cannot link ROMs to their artwork across dataset boundaries, even when permissions are correct on both. The symptom looks like success: the scan finishes, the logs show artwork downloading, the image files are visibly on disk, and every cover in the UI stays blank. The config directory and the database can live on a separate dataset without any issue.
{.is-danger}

> Do **not** `chown -R` the `mysql_data` directory. The MariaDB image manages its own internal UID (999) and sets that path up correctly on first run. That is why the `chown` on `/mnt/tank/configs/romm` above is deliberately not recursive. The `568:568` ownership applies only to the RomM paths.
{.is-info}

> `romm_redis_data` is deliberately a **named volume** rather than a bind mount. RomM 5.x embeds Valkey inside the app container, and this path holds nothing but its cache file: the job queue, task results, and cached lookups. It is fully disposable and rebuilds itself. Docker also seeds a named volume with the image's own ownership, so it works out of the box under `user: 568:568`, while an auto-created bind mount would land as `root:root` and stop Valkey from starting. If you switch it to a bind mount, create and `chown 568:568` the directory yourself first.
{.is-info}

## Custom library structure
Since 5.3 you describe your exact library layout in `config.yml` instead of RomM picking between two fixed layouts. In each pattern, `{platform}` is the platform folder, the final `{game}` is where a game starts, and anything else in `{braces}` is just an organizing folder. You can override the layout per platform, and give a platform a list of layouts when its games sit at more than one depth:

```yaml
filesystem:
    structure:
        default: "roms/{platform}/{game}"
        firmware: "bios/{platform}"
        ps3: "roms/{platform}/{category}/{game}"
        snes: "roms/{platform}/{region}/{game}"
        nes:
            - "roms/{platform}/{game}"
            - "roms/{platform}/{category}/{game}"
```

Platform folder names used by Batocera, RetroBat and ES-DE are recognized out of the box, so a library copied from one of those usually needs no platform mapping at all.

> `filesystem.roms_folder` and `filesystem.firmware_folder` were removed in 5.3. If your `config.yml` still sets either one, RomM refuses to start and prints the equivalent `filesystem.structure` template. Replace the old keys with it.
{.is-warning}

On first launch RomM drops you into a **Setup Wizard** rather than a login screen. The first account you create always receives the Admin role. Give the stack two or three minutes before the web UI answers, since it has to pull images, initialize the database, and run migrations.

# 2 · Reverse Proxy
To put this behind [Nginx Proxy Manager](/nginx), use the following settings:
# {.tabset}
## Details
![screenshot_from_2025-02-24_07-27-52.png](/screenshot_from_2025-02-24_07-27-52.png)
1. Domain Names: romm.example.com (replace example with your own)* Scheme: http
1. Forward Hostname/IP: device IP (like 192.168.X.X)
1. Forward Port: 30061
1. Cache Assets: off
1. Block Common Exploits: on
1. Websockets Support: on ❗
## SSL
![screenshot_from_2025-02-24_07-27-55.png](/screenshot_from_2025-02-24_07-27-55.png)
1. SSL Certificate: "Request a new SSL Certificate"
1. Force SSL: on
1. HTTP/2 Support: on
1. HSTS Enabled: off
1. Email Address for Let's Encrypt: your email address
1. I Agree to the TOS: on
## Advanced
![screenshot_from_2025-02-24_07-28-08.png](/screenshot_from_2025-02-24_07-28-08.png)
Paste this in the box: `proxy_max_temp_file_size 0;`

> Leave **Cache Assets** off. If your proxy caches responses itself, it will serve a stale `index.html` after an upgrade that points at JavaScript bundles the new image no longer ships, which shows up as a blank page or a broken UI.
{.is-warning}

> Once you are serving over HTTPS, set `ROMM_SESSION_SECURE_COOKIE=true` and update `ROMM_BASE_URL` to your public hostname. Invite links are built from `ROMM_BASE_URL`, and RomM will not generate one while it points at `localhost` or another loopback address. HTTPS is also required for OIDC login and for installing RomM as a PWA.
{.is-info}


# 3 · Metadata
RomM will scan without any provider credentials, but match quality drops sharply and some companion app integrations break. Enable at least one. Hasheous is the fastest path since it needs no account at all.

| Provider | Account needed | Notes |
|----------|----------------|-------|
| Hasheous | No | Set `HASHEOUS_API_ENABLED=true`, hash matching out of the box |
| Steam | No | Set `STEAM_API_ENABLED=true`, PC platforms only (`win`, `linux`, `mac`) |
| ScreenScraper | Yes, free | Excellent region-aware artwork |
| IGDB | Yes, via Twitch | Best general metadata and descriptions |
| SteamGridDB | Yes, API key | Community cover art |
| RetroAchievements | Yes, API key | Achievements and progress tracking |
| Demozoo, Pouët, CSDb | No | Demoscene productions, each has its own enable flag |
| MobyGames | Yes, paid | New API keys are behind a paywall |


# {.tabset}
## Hasheous
Hasheous needs no account and no key. Set `HASHEOUS_API_ENABLED=true` in your environment variables and RomM will match ROMs against its hash database on the next scan.

This is the single highest-value provider to enable first, because RomM 5.x matches primarily on file hashes rather than parsing filenames. Correctly named files matter far less than they used to.

If you run your own Hasheous instance, point RomM at it with `HASHEOUS_API_URL` (default `https://hasheous.org/api/v1`).

## Steam
Steam needs no account and no key. Set `STEAM_API_ENABLED=true` and RomM will use the Steam storefront as a metadata source for the `win`, `linux` and `mac` platforms.

Steam also shows up in the **Match ROM** dialog, for when a PC game was missed or matched to the wrong app, and in the cover picker alongside SteamGridDB. To make it the first choice for PC games, add it to your metadata priority in `config.yml`:

```yaml
scan:
    priority:
        metadata:
            - steam # PC platforms only
```

## ScreenScraper
[Create a free ScreenScraper account](https://www.screenscraper.fr/), then set `SCREENSCRAPER_USER` and `SCREENSCRAPER_PASSWORD` to your login credentials. There is no separate API key.

ScreenScraper is region-aware, so a ROM tagged `(Europe)` gets the PAL box art. Since 5.2, scans are paced against your account's reported limits and rejected credentials are reported clearly instead of as a bare 403. You can tune which sources win per artwork type in `config.yml` (in the same file as your `filesystem` block):

```yaml
scan:
    priority:
        cover:
            - igdb
            - ss
        screenshot:
            - ss
            - igdb
        manual:
            - launchbox
```

## IGDB
To access the IGDB API you'll need a Twitch account and a valid phone number for 2FA verification. Up-to-date instructions are available in the [IGDB API documentation](https://api-docs.igdb.com/#account-creation). When registering your application in the Twitch Developer Portal, fill out the form like so:

- Name: Something unique or random like correct-horse-battery-staple or KVV8NDXMSRFJ2MRNPNRSL7GQT
- OAuth Redirect URLs: localhost
- Category: Application Integration
- Client Type: Confidential

> The name you pick has to be unique! Picking an existing name will fail silently, with no error messages. We recommend using `romm-<random hash>`, like `romm-3fca6fd7f94dea4a05d029f654c0c44b`
{.is-info}

Note the client ID and secret that appear on screen, and use them to set `IGDB_CLIENT_ID` and `IGDB_CLIENT_SECRET` in your environment variables.
![screenshot_from_2025-02-25_13-39-41.png](/screenshot_from_2025-02-25_13-39-41.png)
![screenshot_from_2025-02-25_13-39-52.png](/screenshot_from_2025-02-25_13-39-52.png)

## SteamGridDB
To access steamGridDB API, you need to login into their [website](https://www.steamgriddb.com/) with a [steam account](https://store.steampowered.com/join). Once logged in, go to your [API tab under the preferences page](https://www.steamgriddb.com/profile/preferences/api). Copy the key shown and use it to set `STEAMGRIDDB_API_KEY`.

Since 5.1 the manual cover-search dialog has client-side filters, so you can narrow SteamGridDB results instead of scrolling through dozens of community covers. Since 5.2, DMCA-locked grids are skipped automatically.

## RetroAchievements
Log in to [RetroAchievements](https://retroachievements.org/) and grab your API key from the Control Panel under your account settings. Set it as `RETROACHIEVEMENTS_API_KEY`.

RomM will then hash your ROMs against the RA database and surface achievement lists and progress on each game's detail page. To keep progress current automatically, also set `ENABLE_SCHEDULED_RETROACHIEVEMENTS_PROGRESS_SYNC=true`.

## Demoscene
[Demozoo](https://demozoo.org/), [Pouët](https://www.pouet.net/) and [CSDb](https://csdb.dk/) cover demoscene productions on platforms RomM already supports. Enable whichever you want with `DEMOZOO_API_ENABLED=true`, `POUET_API_ENABLED=true` and `CSDB_API_ENABLED=true`.

To match a file, tag its name with the production ID, like `(demozoo-108)`, or paste an ID or production URL into the ROM editor.

## MobyGames
To access the MobyGames API, [create a MobyGames account](https://www.mobygames.com/user/register/) and then visit your profile page. Click the API link under your user name to sign up for an API key. Copy the key shown and use it to set `MOBYGAMES_API_KEY`.

> MobyGames API became a paid feature. Any existing key can be used as usual, but any new API key created will be under a paywall. ScreenScraper is the recommended free alternative.
{.is-warning}

> Want to sort and filter the gallery by how long a game takes to beat? Set `HLTB_API_ENABLED=true`. It uses the HowLongToBeat main story time, so a game needs a scan that matched it on HowLongToBeat before it shows up.
{.is-success}

# 4 · Troubleshooting
# {.tabset}
## General
### Why is not PSP emulation enabled if EmulatorJS supports it?

PSP emulation with the PPSSPP core requires special setup with a reverse proxy, or launching Chrome browser with the `--disable-web-security` and `--enable-features=SharedArrayBuffer` flags, which WE STRONGLY DISCOURAGE as it disables important security features.

Emulator streaming (see section 5) is the better answer for anything EmulatorJS handles poorly.

## Scanning
### Scan is skipping all platforms/ends instantly

There are a few common reasons why a scan may end instantly/without scanning platforms

- Badly mounted library: verify that you mounted your library folder at `/romm/library`
- Incorrect permissions: the app needs to read the files and folders in your library, check their permissions with `ls -lh`
- Wrong library structure: the `filesystem.structure` patterns in `config.yml` must match how your folders are actually laid out on disk (see [Custom library structure](#custom-library-structure))

### ROMs not found for platform X, check romm folder structure

This is the same issue as the one above. Compare your folders against `filesystem.structure` in `config.yml`. With the default layout in this guide, a SNES game belongs at `library/roms/snes/<game>`. If one platform is organized differently from the rest (by region, by category, or at more than one depth), give that platform its own pattern or a list of patterns instead of reorganizing the files.

### Scan does not recognize a platform

The scanner matches each platform folder name against RomM's known platform slugs. Folder names used by Batocera, RetroBat and ES-DE are recognized as aliases, so most libraries work as-is. If a platform still isn't detected, rename the folder to the platform's slug. The slug usually matches the one in the URL of the [platform in IGDB](https://www.igdb.com/platforms): the Nintendo 64DD is https://www.igdb.com/platforms/nintendo-64dd, so the folder would be `nintendo-64dd`.

> Folder names still drive **platform** detection, but individual games are identified by content hash, and on PSX, PS2, PS3, PSP, PS Vita, Switch, 3DS, Wii, Wii U, GameCube, Dreamcast, Xbox and Xbox 360 by the Title ID read from the game's binary. Renamed or moved ROMs keep their saves, favorites, play history and collections instead of being re-imported as duplicates.
{.is-info}

### Scan times out after ~4 hours

The background scan task times out after 4 hours, which can happen if you have a very large library. The easiest work around is to keep running scans every 4 hours, without checking the "Complete re-scan" option. You can also raise `SCAN_TIMEOUT` (in seconds, default `14400`) in your environment variables. Since 5.3 RomM runs 4 scan workers by default (`SCAN_WORKERS`), so large libraries finish much faster than before.

### Artwork downloads but covers stay blank

Your `library` and `resources` volumes are on different ZFS datasets. See the warning in section 1. Move them onto the same dataset and rescan.

## Authentication Issues
### Error: 403 Forbidden

When authentication is enabled, most endpoints will return a `403 Forbidden` response if you're not authenticated, or if your sessions is in a broken state. The session key can be reset by [clearing your cookies](https://support.google.com/accounts/answer/32050).

CSRF protection is also enabled, which helps to mitigates [CSRF attacks](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html) (useful if your instance is public). If you encounter a `Forbidden (403) CSRF verification failed error`, simply reloading your browser should force it to fetch a fresh CSRF cookie.

### Error: Unable to login: CSRF token verification failed

This error is known to happen on Chrome, but could happen in other browsers; manually clear your cookies (specifically one called `csrftoken`) and hard reload your browser window (<kbd>CMD+SHIFT+R</kbd> on macOS, <kbd>CTRL+F5</kbd> on Windows).

### Error: 400 Bad Request on the Websocket endpoint

If you're running RomM behind a reverse-proxy (Caddy, Nginx, etc.), ensure that Websockets are supported and enabled. This may vary depending on the reverse proxy solution being used. In the case of Nginx Proxy Manager, enable the "Websockets Support" toggle when editing the proxy host.

## Upgrading
### RomM refuses to start after upgrading to 5.3

5.3 no longer detects your library layout. Add a `filesystem.structure` block to `config.yml` (see section 1) and restart. If your old `config.yml` sets `filesystem.roms_folder` or `filesystem.firmware_folder`, remove them; RomM prints the equivalent `filesystem.structure` template in the logs to copy from.

### Emulator streaming stopped working after upgrading to 5.3

Streaming config changed from one entry per emulator to one entry per container. Existing setups have to rewrite that part of `config.yml` before streaming comes back up. See section 5.

### Blank page or broken UI right after an upgrade

Your reverse proxy or CDN cached the old `index.html`, which points at JavaScript bundles the new image no longer ships. Purge the proxy cache once and hard reload. Turning **Cache Assets** off in NPM prevents it recurring.

### Migrations fail on MariaDB with binary logging enabled

When binlog is on and the connecting user lacks `SUPER`, MariaDB refuses the trigger DDL that RomM's migrations need. Run this as an admin or root user, not as the romm app user:

```sql
SET GLOBAL log_bin_trust_function_creators = 1;
```

`SET GLOBAL` is lost on database restart. Make it permanent by adding this under `[mysqld]` in `my.cnf`:

```
log_bin_trust_function_creators = 1
```

Or grant the privilege to the app user on MariaDB 10.5+:

```sql
GRANT BINLOG ADMIN ON *.* TO 'romm'@'%';
```

> A fresh deployment from the compose file above will not hit this. It only affects existing shared MariaDB instances.
{.is-info}

## Miscellaneous

### Error: Could not get twitch auth token: check client_id and client_secret

This is likely due to mis-configured environment variables; verify that `IGDB_CLIENT_ID` and `IGDB_CLIENT_SECRET` are set correctly, and that both match the values in the Twitch Developer Portal.

# 5 · Emulator Streaming
RomM 5.1 introduced emulator streaming, which launches a game into a **native** emulator running in a separate container and streams the picture, sound and input back to your browser. Unlike EmulatorJS, emulation runs server-side on real binaries such as PCSX2, Dolphin and Xemu, so your server does the heavy lifting rather than the client. That opens up systems EmulatorJS handles poorly, including PS2, GameCube and original Xbox.

It requires the `STREAMING_BROKER_SECRET` environment variable plus separate broker containers.

RomM 5.3 reworked streaming substantially:

- Config is **one entry per container**, not one per emulator. A container's host, protocol and label become the defaults for every platform it serves, and a platform block only overrides what differs.
- A platform can pool several containers and grab whichever one is free.
- Platforms that support both browser and streamed play get separate **Play** and **Stream** buttons.
- Saves and save states made while streaming sync back into your library with history and thumbnails.
- Multi-disc games get a disc-swap control, and PS2 and GameCube can sync a whole memory card.

Official setup docs are now available: [Emulator Streaming](https://docs.romm.app/5.3.0-beta/using/emulator-streaming/).

> Emulator streaming is still maturing and moves quickly between releases. Treat it as experimental, and read the release notes before every upgrade if you depend on it.
{.is-warning}

# <img src="/patreon-light.png" class="tab-icon"> 6 · Video
[![](/2026-08-06-romm-51-the-self-hosted-rom-ma-promo-card.png)](https://www.patreon.com/serversathome/posts/romm-5-1-self-165952710)