---
title: File Browser
description: A guide to deploy the File Browser Quantum replacement in docker
published: true
date: 2026-10-15T10:28:02.565Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:39.442Z
---

# <img src="/filebrowser-quantum.png" class="tab-icon"> What is File Browser Quantum?

**File Browser Quantum** is a self-hosted, web-based file manager: browse, upload, download, preview, edit, search, and share the files on your server from any browser. It's called "Quantum" because it packs a huge amount of functionality (user management, SSO, indexed search, rich media previews) into a single tiny binary that's genuinely easy to run.

If you want one tool to put a clean, fast web UI in front of a folder (or your whole pool), this is my pick.

<div class="glance">
  <div><span>Port</span><b><code>7999</code></b></div>
  <div><span>Deploy via</span><b>TrueNAS app or Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty intermediate">Intermediate</b></div>
  <div class="glance-links">
    <a href="https://github.com/gtsteffaniak/filebrowser"><i class="mdi mdi-github"></i>Project</a>
    <a href="https://filebrowserquantum.com/en/docs/"><i class="mdi mdi-book-open-variant"></i>Docs</a>
    <a href="https://apps.truenas.com/catalog/filebrowser-quantum_community/"><i class="mdi mdi-server"></i>TrueNAS app</a>
  </div>
</div>

> **Version 2.0 is here, and it's a breaking upgrade.** v2.0.0-stable (October 2026) moved the database to SQLite and restructured the config. If you were already running 1.x on the `:stable` tag, your container may have pulled 2.0 and refused to start. Jump to [5 · Upgrading from 1.x](#h-5-upgrading-from-1x) before anything else.
{.is-warning}

# <img src="/docker.png" class="tab-icon"> 1 · Deploy File Browser


### Create your config

Quantum is driven by a `config.yaml` file. Create it **before** the first launch, inside the folder you'll mount as the data volume, so `/mnt/tank/configs/filebrowser/config.yaml`. Here's a solid single-source starting point for v2:

```yaml
server:
  cacheDir: /home/filebrowser/data/tmp   # keep generated thumbnails on the data volume
  logging:
    - levels: "info|warning|error"
  sources:
    - path: "/srv"                        # maps to /mnt below, your whole pool
      name: "Pool"
      config:
        defaultEnabled: true              # auto-add this source for new users
        defaultPermissions:               # what new users can do on this source
          view: true
          download: true
          create: true
          modify: true
          delete: true
http:
  port: 80
  baseURL: "/"
userDefaults:
  ui:
    darkMode: true
  listing:
    singleClick: true
  preview:
    image: true
    video: false
    office: false
    popup: true
  account:
    permissions:
      api: true
      share: true
```

> **`userDefaults` is the template applied to new users, not your admin account.** Be deliberate here. I left `admin: true` *out* on purpose: if you ever enable signups or add a second person, you don't want everyone minted as an admin automatically. Grant admin explicitly to the accounts that need it instead.
>
> In v2, file permissions (view, download, create, modify, delete) are **per source** via `defaultPermissions`, while `userDefaults.account.permissions` only covers account-level rights like `api`, `share` and `admin`. Any field you set under `userDefaults` is also **locked** in the admin UI (Settings → User defaults). If you'd rather manage defaults from the UI, leave the whole `userDefaults` block out.
{.is-info}

> Quantum **strictly validates** the config. An unknown key (a typo, or an option that moved in a newer release) stops the container from starting rather than being silently ignored. If a deploy fails right away, the logs will name the offending field. The most common v2 offender: `port`, `baseURL` and `externalUrl` now live under `http:`, not `server:`.
{.is-warning}

Everything you can set is in the [full config reference](https://filebrowserquantum.com/en/docs/reference/fullconfig/) and the [annotated default config](https://github.com/gtsteffaniak/filebrowser/blob/main/frontend/public/config.generated.yaml).

### Deploy the container

```yaml
services:
  filebrowser:
    image: ghcr.io/gtsteffaniak/filebrowser:2.0-stable
    container_name: filebrowser
    user: "568:568"
    volumes:
      - /mnt:/srv
      - /mnt/tank/configs/filebrowser:/home/filebrowser/data
    ports:
      - 7999:80
    restart: unless-stopped
```

Before the first start, give the apps user ownership of the config folder:

```bash
chown -R 568:568 /mnt/tank/configs/filebrowser
```

> The image auto-detects its config at `/home/filebrowser/data/config.yaml`, which is exactly where the volume above places it, so no environment variable is needed. The SQLite database lands on that same volume.
>
> **Why `2.0-stable` and not `:stable` or `:latest`:** Quantum hard-fails on config it doesn't recognise, and the jump from 1.x to 2.0 proved that a floating tag can hand you a container that won't boot after an unattended pull. `2.0-stable` still picks up every 2.0.x patch (including security fixes) but won't move you to 2.1 until you change the tag yourself. If you want a ~15 MB core-only container with no media or office previews, use the slim variant instead; the [Docker docs](https://filebrowserquantum.com/en/docs/getting-started/docker/) list the current slim tags.
{.is-info}

### Who the container runs as

The image's built-in default is a `filebrowser` user with UID:GID `1000:1000`. I override that with `user: "568:568"`, the TrueNAS **apps** user, so FileBrowser owns its config the same way every other app on the box does.

> **Know what that means for a pool-wide mount.** A non-root UID can only change the files *that* UID can write. With your whole pool mounted at `/srv`, FileBrowser will happily *list* everything but will fail to move, rename, or delete files owned by other users (media datasets, SMB shares, older app data). Two clean ways to handle it:
>
> - Mount only the datasets you actually manage from the browser (for example `/mnt/tank/media:/srv/media`) and make sure those are owned by, or ACL-writable for, the apps user (568).
> - Mark sources you only want to browse as `readOnly: true` in their `config` so nobody is surprised by permission errors.
>
> Resist the urge to "fix" this by running as root (`user: "0:0"`). A root container holds the keys to your entire mounted pool, and Quantum has shipped serious path-traversal bugs (see section 2). Fix the file ownership instead.
{.is-warning}



# 2 · First login & hardening

**There is no `admin` / `admin` anymore.** On a fresh v2 Docker install with no admin password set, Quantum generates a random one (it looks like `word-xxxxx-xx`) and prints it **once** in the container log. Grab it with:

```bash
docker logs filebrowser 2>&1 | grep -i password
```

Log in as `admin` with that password and you'll be forced to set a new one immediately. On TrueNAS, use the admin password you set in the install form.

> Prefer to set it yourself? Add `auth.methods.password.adminPassword: "something-strong"` to `config.yaml`. Note that a password set there is **re-applied on every startup**, so remove it after first login if you plan to change the password from the UI.
{.is-info}

Once you're in, a couple of things worth doing on day one: turn on TOTP (or the new **passkey** MFA) for your account, review which sources and permissions new users inherit, and, if this is exposed beyond your LAN, read the SSO and reverse-proxy tabs below before opening it up.

> **Stay on a current release, especially if this is internet-facing.** Recent security fixes include:
> - **2.0.1**: a High-severity bug where creating an account with `..` in the username could escape its scoped folder, and a session-token collision when several users logged in at once.
> - **2.0.0**: public-share subtitle routes ignoring share download limits, OIDC callback tampering, and open redirects on login/logout.
> - **1.5.x**: High-severity symlink escape, share/upload access-control bypasses, and stored XSS via HTML preview.
>
> Run **2.0.1 or newer** and update deliberately. Advisories are listed on the [GitHub security page](https://github.com/gtsteffaniak/filebrowser/security/advisories).
{.is-danger}

# 3 · Configuration

# {.tabset}
## Multiple sources

The headline feature. Mount more than one location and give each a name. Quantum indexes and searches them independently:

```yaml
server:
  sources:
    - path: "/srv/media"
      name: "Media"
      config:
        defaultEnabled: true
        readOnly: true            # browse and download only, from UI, WebDAV and API
    - path: "/srv/documents"
      name: "Documents"
      config:
        defaultEnabled: true
        defaultPermissions:
          view: true
          download: true
          create: true
          modify: true
          delete: false           # new users can't delete here
```

v2 gives you two levers: `readOnly: true` locks a source for **everyone**, while `defaultPermissions` sets what **new users** get on that source (and can be tuned per user later under User Management). See [Sources](https://filebrowserquantum.com/en/docs/configuration/sources/) and [Access Control](https://filebrowserquantum.com/en/docs/access-control/access-control-overview/).

## Single sign-on (OIDC)

Drop sign-in into your existing identity provider (Authentik, Authelia, Pocket ID, Google, etc.):

```yaml
auth:
  methods:
    oidc:
      enabled: true
      clientId: "filebrowser"
      clientSecret: "your-client-secret"
      issuerUrl: "https://auth.example.com/application/o/filebrowser/"
      scopes: "openid email profile"
      userIdentifier: "preferred_username"
      groupsClaim: "groups"
      adminGroup: "filebrowser-admins"   # members of this group get admin
```

v2 config also expands environment variables, so you can write `clientSecret: "${OIDC_SECRET}"` and pass the secret in through the compose `environment:` block instead of keeping it in the file. Full walkthrough (including LDAP, JWT, and proxy auth) is in the [authentication docs](https://filebrowserquantum.com/en/docs/configuration/authentication/oidc/).

## Enforce 2FA

Require TOTP for every password user instead of leaving it opt-in:

```yaml
auth:
  methods:
    password:
      enabled: true
      enforcedOtp: true
      minLength: 8
      signup: false   # keep signups off unless you really want open registration
```

## Behind a reverse proxy

If you're fronting it with Cloudflare Tunnel, Traefik, NPM, Caddy, etc., all of these settings live under `http:` in v2:

- On a **subpath** (e.g. `example.com/files`), set `http.baseURL: "/files"`.
- On a **subdomain** (most tunnel setups), leave `baseURL: "/"`.
- For **share links to use your public domain** instead of an internal address, set `http.externalUrl`.
- Set `http.trustProxyHeaders: true` so FileBrowser sees the real client IP (it's `false` by default, which breaks rate limiting and logging behind a proxy).

```yaml
http:
  port: 80
  baseURL: "/"
  externalUrl: "https://files.example.com"   # public base URL used in share links
  trustProxyHeaders: true
```

> Putting `externalUrl` or `baseURL` under `server:` (where they lived in 1.x) or under a source's `config` will fail strict validation and stop the container from starting.
{.is-warning}

Details and proxy-header notes: [running behind a reverse proxy](https://filebrowserquantum.com/en/docs/getting-started/reverse-proxy/).

# 4 · Quantum vs. the original File Browser

The original File Browser (`filebrowser/filebrowser`) is **archived as of September 2026**. The maintainers have said there will be no further releases, bug fixes, or security fixes, and recommend keeping it off the public internet. TrueNAS has marked its old app as deprecated too. [gtsteffaniak](https://github.com/gtsteffaniak/filebrowser) forked it into **Quantum**, which is actively maintained and is now on its **v2.0** stable line. This is the one to run.

> **What Quantum adds over the original:** multiple sources at once, login via OIDC / LDAP / JWT / password + TOTP or passkeys, ultra-fast SQLite indexed search that updates in real time, thumbnails for office docs / video / album art / 3D models, granular sharing (expiry, anonymous access, download limits, per-share permissions), per-source access control, user activity logs, and long-lived API tokens with a built-in Swagger page. The one thing it removed on purpose: shell commands.
{.is-success}

| Feature                    | Quantum            | Original File Browser | Nextcloud           |
| -------------------------- | ------------------ | --------------------- | ------------------- |
| Actively developed         | ✅ stable releases  | ❌ archived, no fixes  | ✅                   |
| Multiple folders / sources | ✅                  | ❌                     | ✅                   |
| Indexed real-time search   | ✅                  | ❌                     | add-on              |
| SSO (OIDC / LDAP)          | ✅                  | ❌                     | ✅                   |
| API tokens + docs page     | ✅                  | ❌                     | ✅                   |
| Office / video previews    | ✅                  | ❌                     | ✅                   |
| Footprint                  | tiny single binary | tiny                  | heavy full platform |

📌 Full official docs live at [filebrowserquantum.com](https://filebrowserquantum.com/).

# 5 · Upgrading from 1.x

v2.0 needs a **database migration** (BoltDB to SQLite) and **config changes**. If your container stopped starting after an update, this is almost certainly why: v2 refuses to boot while an un-renamed `database.db` sits in the data folder.

1. **Stop the container** and back up the data folder:
   ```bash
   docker stop filebrowser
   cp -a /mnt/tank/configs/filebrowser /mnt/tank/configs/filebrowser-v1-backup
   ```
2. **Rename the old database** so v2 will start:
   ```bash
   mv /mnt/tank/configs/filebrowser/database.db /mnt/tank/configs/filebrowser/database.db.old
   ```
3. **Convert your config.** Paste your old `config.yaml` into the [config migration tool](https://filebrowserquantum.com/tools/config-migration/) (it runs entirely in your browser, nothing is uploaded). It moves the HTTP keys under `http:`, converts `userDefaults` to the new layout, and translates `trustedHeaders`. If you use a reverse proxy, confirm `http.trustProxyHeaders: true` is set.
4. **Point v2 at the old database** by adding this to the converted config:
   ```yaml
   server:
     database:
       migrateFrom: "database.db.old"
   ```
5. **Update the image tag** to `ghcr.io/gtsteffaniak/filebrowser:2.0-stable` and start it. Watch `docker logs -f filebrowser` for the "Migration completed successfully" line. Users, passwords, shares, access rules, API tokens and sidebar links all carry over.
6. **Verify** everyone can log in, shares still resolve, and the per-source permissions under User Management look right. Your old global permissions get copied onto every source a user has, so double-check anyone who should only modify or delete in one place.
7. **Clean up:** remove the `migrateFrom` line, restart, and keep `database.db.old` somewhere safe outside the data folder until you're sure you won't roll back.

> If you set the database location with the `FILEBROWSER_DATABASE` environment variable, rename it to `FILEBROWSER_DATABASE_PATH`. The old variable now **stops startup**. Anything that called the old API (`/api/raw`, `PUT /api/users`, the `X-Renew-Token` header) also needs updating; see the [v2 release notes](https://github.com/gtsteffaniak/filebrowser/releases/tag/v2.0.0-stable) and the [full migration guide](https://filebrowserquantum.com/en/docs/getting-started/v2/migration/).
{.is-info}

> **Rolling back:** stop the container, restore `/mnt/tank/configs/filebrowser-v1-backup`, and pin the image to your last 1.5.x stable tag.
{.is-warning}

# <img src="/youtube.png" class="tab-icon"> 6 · Video

<!-- Paste the FileBrowser video URL here once it's published, and add it to the glance panel links -->