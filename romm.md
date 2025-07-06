---
title: RomM
description: A guide to deploying RomM on TrueNAS and Docker
published: true
date: 2025-07-06T10:11:49.563Z
tags: 
editor: markdown
dateCreated: 2025-02-24T11:29:26.783Z
---

![romm.png](/romm.pn

![screenshot_from_2025-02-24_06-28-38.png](/screenshot_from_2025-02-24_06-28-38.png)
# What is RomM?
RomM (ROM Manager) allows you to scan, enrich, browse and play your game collection with a clean and responsive interface. With support for multiple platforms, various naming schemes, and custom tags, RomM is a must-have for anyone who plays on emulators.

> See the full documentation [here](https://docs.romm.app/latest/)
{.is-info}


# Installation
# {.tabset}
## TrueNAS
![screenshot_from_2025-02-24_06-38-49.png](/screenshot_from_2025-02-24_06-38-49.png)

1. Set a secure database password
1. Set a secure redis password
1. Generate an auth secret key using `openssl rand -hex 32` in the terminal
1. Input any client IDs or API keys you have (optional)
1. Set **Host Path** for the library storage, resources storage, config storage, assets storage, and Postgres Data Storage

> When setting the Host Path for Postgres Data Storage be sure to check the box for **Automatic Permissions**!
{.is-warning}
6. Increase Resource limits (optional)

## Docker Compose
```yaml
services:
    romm:
        image: rommapp/romm:latest
        container_name: romm
        restart: unless-stopped
        user: 568:568
        environment:
            - DB_HOST=romm-db
            - DB_NAME=romm
            - DB_USER=romm-user
            - DB_PASSWD=dbpassword
            - ROMM_AUTH_SECRET_KEY=03a054b6ca27e0107c5eed552ea66becd9f3a2a8a91e7595cd462a593f9ecd09 # Generate a key with `openssl rand -hex 32`
            - IGDB_CLIENT_ID= # Generate an ID and SECRET in IGDB
            - IGDB_CLIENT_SECRET= # https://api-docs.igdb.com/#account-creation
            - MOBYGAMES_API_KEY= # https://www.mobygames.com/info/api/
            - STEAMGRIDDB_API_KEY= # https://github.com/rommapp/romm/wiki/Generate-API-Keys#steamgriddb
        volumes: 
            - /mnt/tank/configs/romm/resources:/romm/resources
            - /mnt/tank/configs/romm/romm_redis_data:/romm/redis-data
            - /mnt/tank/configs/romm/assets:/romm/assets
            - /mnt/tank/configs/romm/config:/romm/config
            - /mnt/tank/roms:/romm/library # replace with your romms library path
        ports:
            - 30061:8080
        depends_on:
            romm-db:
                condition: service_healthy
                restart: true

    romm-db:
        image: mariadb:latest
        container_name: romm-db
        restart: unless-stopped
        environment:
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


# Reverse Proxy
To put this behind [Ngnix Reverse Proxy Manager](/nginx), use the following settings:
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


# Metadata
# {.tabset}
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

## MobyGames
To access the MobyGames API, [create a MobyGames account](https://www.mobygames.com/user/register/) and then visit your profile page. Click the API link under your user name to sign up for an API key. Copy the key shown and use it to set `MOBYGAMES_API_KEY`.

> MobyGames API became a paid feature. Any existing key can be used as usual, but any new API key created will be under a paywall
{.is-warning}

## SteamGridDB

To access steamGridDB API, you need to login into their [website](https://www.steamgriddb.com/) with a [steam account](https://store.steampowered.com/join). Once logged in, go to your [API tab under the preferences page](https://www.steamgriddb.com/profile/preferences/api). Copy the key shown and use it to set `STEAMGRIDDB_API_KEY`.

# Troubleshooting
# {.tabset}
## General
### Why is not PSP emulation enabled if EmulatorJS supports it?

PSP emulation with the PPSSPP core requires special setup with a reverse proxy, or launching Chrome browser with the `--disable-web-security` and `--enable-features=SharedArrayBuffer` flags, which WE STRONGLY DISCOURAGE as it disables important security features.

## Scanning
### Scan is skipping all platforms/ends instantly

There are a few common reasons why a scan may end instantly/without scanning platforms

- Badly mounted library: verify that you mounted your ROMs folder at `/romm/library`
- Incorrect permissions: the app needs to read the files and folders in your library, check their permissions with ls -lh
- Invalid folder structure: verify that your folder structure matches the one in the README

### ROMs not found for platform X, check romm folder structure

This is the same issue as the one above, and can be quickly solved by verifying your folder structure. RomM expects a library with a folder named roms in it, for example:

- `/server/media/library:/romm/library`
- `/server/media/games/roms:/romm/library/roms`

### Scan does not recognize a platform

When scanning the folders mounted in `/library/roms`, the scanner tries to match the folder name with the platform's slug in IGDB. If you notice that the scanner isn't detecting a platform, verify that the folder name matches the slug in the URL of the [platform in IGDB](https://www.igdb.com/platforms). For example, the Nintendo 64DD has the URL https://www.igdb.com/platforms/nintendo-64dd, so the folder should be named `nintendo-64dd`.

### Scan times out after ~4 hours

The background scan task times out after 4 hours, which can happen if you have a very large library. The easiest work around is to keep running scans every 4 hours, without checking the "Complete re-scan" option.

## Authentication Issues
### Error: 403 Forbidden

When authentication is enabled, most endpoints will return a `403 Forbidden` response if you're not authenticated, or if your sessions is in a broken state. The session key can be reset by [clearing your cookies](https://support.google.com/accounts/answer/32050).

CSRF protection is also enabled, which helps to mitigates [CSRF attacks](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html) (useful if your instance is public). If you encounter a `Forbidden (403) CSRF verification failed error`, simply reloading your browser should force it to fetch a fresh CSRF cookie.

### Error: Unable to login: CSRF token verification failed

This error is known to happen on Chrome, but could happen in other browsers; manually clear your cookies (specifically one called `csrftoken`) and hard reload your browser window (<kbd>CMD+SHIFT+R</kbd> on macOS, <kbd>CTRL+F5</kbd> on Windows).

### Error: 400 Bad Request on the Websocket endpoint

If you're running RomM behind a reverse-proxy (Caddy, Nginx, etc.), ensure that Websockets are supported and enabled. This may vary depending on the reverse proxy solution being used. In the case of Nginx Proxy Manager, enable the "Websockets Support" toggle when editing the proxy host.

## Miscellaneous

### Error: Could not get twitch auth token: check client_id and client_secret

This is likely due to mis-configured environment variables; verify that `CLIENT_ID` and `CLIENT_SECRET` are set correctly, and that both match the values in IGDB.

# Video Walkthrough
[](https://youtu.be/lQeUq5Pzo1o)