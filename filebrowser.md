---
title: File Browser
description: A guide to deploy the File Browser Quantum replacement in docker
published: true
date: 2026-10-15T10:31:10.137Z
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



Everything you can set is in the [full config reference](https://filebrowserquantum.com/en/docs/reference/fullconfig/) and the [annotated default config](https://github.com/gtsteffaniak/filebrowser/blob/main/frontend/public/config.generated.yaml).

### Deploy the container

```yaml
services:
  filebrowser:
    image: ghcr.io/gtsteffaniak/filebrowser:2.0-stable
    container_name: filebrowser
    user: "0:0"
    volumes:
      - /mnt:/srv
      - /mnt/tank/configs/filebrowser:/home/filebrowser/data
    ports:
      - 7999:80
    restart: unless-stopped
```


# 2 · First login & hardening

**There is no `admin` / `admin` anymore.** On a fresh v2 Docker install with no admin password set, Quantum generates a random one (it looks like `word-xxxxx-xx`) and prints it **once** in the container log. Grab it with:

```bash
docker logs filebrowser 2>&1 | grep -i password
```

Log in as `admin` with that password and you'll be forced to set a new one immediately. 


Once you're in, a couple of things worth doing on day one: turn on TOTP (or the new **passkey** MFA) for your account, review which sources and permissions new users inherit, and, if this is exposed beyond your LAN, read the SSO and reverse-proxy tabs below before opening it up.


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
