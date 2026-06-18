---
title: File Browser
description: A guide to deploy the File Browser Quantum replacement in docker
published: true
date: 2026-06-18T17:50:17.380Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:39.442Z
---

# <img src="/filebrowser-quantum.png" class="tab-icon"> What is File Browser Quantum?

File Browser Quantum is a self-hosted, web-based file manager: browse, upload, download, preview, edit, search, and share the files on your server from any browser. It's called "Quantum" because it packs a huge amount of functionality — user management, SSO, indexed search, rich media previews — into a single tiny binary that's genuinely easy to run.

If you want one tool to put a clean, fast web UI in front of a folder (or your whole pool), this is my pick.

# Quantum vs. the original File Browser

The original File Browser (`filebrowser/filebrowser`) still works, but it's in [maintenance-only mode](https://github.com/filebrowser/filebrowser/discussions/4906#discussioncomment-13436994) — no new features. [gtsteffaniak](https://github.com/gtsteffaniak/filebrowser) forked it into **Quantum**, and it's now a properly maintained project with stable releases (the stable line is now **v1.4.x**). This is the one I'd run today.

> **What Quantum adds over the original:** multiple sources at once, login via OIDC / LDAP / JWT / password + 2FA, ultra-fast SQLite indexed search that updates in real time, thumbnails for office docs / video / album art / 3D models, granular sharing (expiry, anonymous access, per-share permissions), directory-level access control, and long-lived API tokens with a built-in Swagger page. The one thing it removed on purpose: shell commands.
{.is-info}

| Feature                    | Quantum            | Original FileBrowser | Nextcloud           |
| -------------------------- | ------------------ | -------------------- | ------------------- |
| Actively developed         | ✅ stable releases  | ⚠️ maintenance only  | ✅                   |
| Multiple folders / sources | ✅                  | ❌                    | ✅                   |
| Indexed real-time search   | ✅                  | ❌                    | add-on              |
| SSO (OIDC / LDAP)          | ✅                  | ❌                    | ✅                   |
| API tokens + docs page     | ✅                  | ❌                    | ✅                   |
| Office / video previews    | ✅                  | ❌                    | ✅                   |
| Footprint                  | tiny single binary | tiny                 | heavy full platform |


📌 Full official docs now live at [filebrowserquantum.com](https://filebrowserquantum.com/).

# 1 · Create your config

Quantum is driven by a `config.yaml` file. Create it **before** the first launch, inside the folder you'll mount as the data volume — so `/mnt/tank/configs/filebrowser/config.yaml`. Here's a solid single-source starting point:

```yaml
server:
  port: 80
  baseURL: "/"
  cacheDir: /home/filebrowser/data/tmp   # keep generated thumbnails on the data volume
  logging:
    - levels: "info|warning|error"
  sources:
    - path: "/srv"                        # maps to /mnt below — your whole pool
      name: "Pool"
      config:
        defaultEnabled: true              # auto-add this source for new users
userDefaults:
  darkMode: true
  singleClick: true
  preview:
    image: true
    video: false
    office: false
    popup: true
  permissions:
    modify: true
    share: true
    api: true
```

> **`userDefaults` is the template applied to every new user — not your admin account.** Be deliberate here. I left `admin: true` *out* on purpose: if you ever enable signups or add a second person, you don't want everyone minted as an admin automatically. Grant admin explicitly to the accounts that need it instead.
{.is-warning}

> Quantum **strictly validates** the config. An unknown key — a typo, or an option removed in a newer release — stops the container from starting rather than being silently ignored. If a deploy fails right away, the logs will name the offending field. (This is also why the old `preview.highQuality` key no longer works.)
{.is-warning}

Everything you can set is in the [full config reference](https://filebrowserquantum.com/en/docs/reference/fullconfig/) and the [annotated default config](https://github.com/gtsteffaniak/filebrowser/blob/main/frontend/public/config.generated.yaml).

# <img src="/docker.png" class="tab-icon"> 2 · Deploy File Browser

```yaml
services:
  filebrowser:
    image: gtstef/filebrowser:stable
    container_name: filebrowser
    user: "0:0"
    volumes:
      - /mnt:/srv
      - /mnt/tank/configs/filebrowser:/home/filebrowser/data
    ports:
      - 7999:80
    restart: unless-stopped
```

> The image auto-detects its config at `/home/filebrowser/data/config.yaml`, which is exactly where the volume above places it — no environment variable needed. The default database lands at `/home/filebrowser/data/database.db` on that same volume. The `:stable` tag bundles FFmpeg for video/office thumbnails; swap to `:stable-slim` for a ~15 MB core-only image with no media previews. I use `:stable` over `:latest` deliberately — because Quantum hard-fails on config it doesn't recognise and minor versions can introduce breaking changes, you don't want an unattended pull surprising you with a container that won't boot.
{.is-info}

## 2.1 Who the container runs as (read this before first start)

**Since v1.3.0 the image no longer runs as `root` by default — it runs as a built-in `filebrowser` user with UID:GID `1000:1000`.** (On v1.2.x and earlier it ran as root.) That non-root default is a deliberate hardening step by the maintainer, and for most setups it's the right call: pick a UID, `chown` your data to match, done.

I run mine as **root** instead — `user: "0:0"` in the compose above — and you should understand exactly why before you copy that line.

**Why root here:** my pool is mounted whole at `/srv`, and the files across it are owned by a grab-bag of UIDs — media datasets, app data, shares created by different users over the years. A single non-root UID can only write the files *that* UID owns, so it would happily *list* everything but silently fail to move, rename, or delete anything owned by someone else. Root sidesteps all of it: it reads and writes every file regardless of owner, which is the whole point of a pool-wide file manager for me. As a bonus there's no `chown` dance on the config volume — root can always write its own database.

> **This is a real security tradeoff, not a free shortcut.** A root container holds the keys to your entire mounted pool. If FileBrowser is ever compromised — a malicious share, an unpatched CVE (and Quantum has shipped serious ones; see the hardening note above), a container escape — the attacker inherits root's reach: read, modify, encrypt, or delete **any** file under `/srv`, not just one user's data. You're trading the maintainer's non-root hardening for convenience. Only accept that if **all** of the following are true:
> - the instance is **not** exposed raw to the internet — keep it LAN-only, or behind SSO / a tunnel that enforces auth,
> - you stay on a **current, patched** release (**≥ 1.4.1**), and
> - you trust every account that can log in.
>
> If any of those isn't true, don't run as root. Match the `user:` UID to the data you actually need to manage instead — `user: "568:568"` for TrueNAS apps-owned data, or `"1000:1000"` with `chown -R 1000:1000 /mnt/tank/configs/filebrowser` — and fix the underlying file ownership rather than papering over it with root.
{.is-danger}

# 3 · First login & hardening

The default user is `admin` and the default password is `admin`.

> **Change the admin password the moment you log in** (top-right user menu → **Settings**). Default credentials on a reachable file manager is the fastest way to lose everything on the box.
{.is-danger}

Once you're in, a couple of things worth doing on day one: turn on TOTP for your account, review which sources and permissions new users inherit, and — if this is exposed beyond your LAN — read the SSO and reverse-proxy tabs below before opening it up.

> **Run a current release if this is internet-facing.** The 1.4.x line patched a *critical* unauthenticated path-traversal in public-share delete (arbitrary file deletion) and a stored XSS via SVG in public shares, plus a 1.4.0 regression that exposed source information on shares (fixed in **1.4.1**). If you expose shares through a tunnel or proxy, stay on **≥ 1.4.1** and update deliberately.
{.is-danger}

# 4 · Going further {.tabset}

## Multiple sources

The headline feature. Mount more than one location and give each a name — Quantum indexes and searches them independently:

```yaml
server:
  sources:
    - path: "/srv/media"
      name: "Media"
      config:
        defaultEnabled: true
    - path: "/srv/documents"
      name: "Documents"
      config:
        defaultEnabled: true
```

Whether a user can write, delete, or only read a source is governed by their **permissions**, not a per-source flag — so a "read-only" share is just a user/group without `modify`, `create`, or `delete`. See [Sources](https://filebrowserquantum.com/en/docs/configuration/sources/) and [Access Control](https://filebrowserquantum.com/en/docs/access-control/access-control-overview/).

## Single sign-on (OIDC)

Drop sign-in into your existing identity provider (Authentik, Authelia, Google, etc.):

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
      adminGroup: "filebrowser-admins"   # members of this group get admin
```

Full walkthrough (including LDAP, JWT, and proxy auth) is in the [authentication docs](https://filebrowserquantum.com/en/docs/configuration/authentication/oidc/).

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

If you're fronting it with Cloudflare Tunnel, Traefik, NPM, etc.:

- On a **subpath** (e.g. `example.com/files`), set `server.baseURL: "/files"`.
- On a **subdomain** (most tunnel setups), leave `baseURL: "/"`.
- For **share links to use your public domain** instead of an internal address, set `externalUrl` at the **server** level. It is *not* a per-source `config` key — putting it under a source will fail Quantum's strict validation and stop the container from starting:

```yaml
server:
  externalUrl: "https://files.example.com"   # public base URL used in share links
  sources:
    - path: "/srv"
      name: "Pool"
      config:
        defaultEnabled: true
```

Details and proxy-header notes: [running behind a reverse proxy](https://filebrowserquantum.com/en/docs/getting-started/reverse-proxy/).