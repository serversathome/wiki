---
title: openGym
description: A guide to deploy openGym
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-08-05T20:35:06.941Z
---

# What is openGym?

**openGym** is a self-hosted gym and body-weight tracker — a fully private replacement for Strong, Hevy, or the fitness app that came with your phone. You plan a routine for each day of the week, run guided workouts that pre-fill your weights from last session, and track body weight against a goal line. It ships with a library of **1,324 exercises** complete with animated demos, supports supersets and cardio logging, detects PRs, and draws a GitHub-style activity heatmap of your training year.

It installs to your home screen as a PWA, signs you in with **passkeys** (Face ID / Touch ID / fingerprint) instead of passwords, and syncs across your phone and laptop. There is no database server, no cloud dependency, and **no telemetry** — everything lives as plain JSON files in a folder you control.

Stack is React 19 + Vite on the front, a dependency-light Node API on the back, nginx in front of both. Licensed AGPL v3.

<div class="glance">
  <div><span>Port</span><b><code>8080</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>2 services</b></div>
  <div><span>Difficulty</span><b class="difficulty intermediate">Intermediate</b></div>
  <div class="glance-links">
    <a href="https://github.com/DuarteSantos8/openGym"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>


# <img src="/docker.png" class="tab-icon"> 1 · Deploy openGym


```yaml
services:

  # ── API: passkey auth + per-user data ───────────────────────────────────
  opengym-api:
    image: ghcr.io/duartesantos8/opengym-api:latest
    container_name: opengym-api
    user: "568:568"
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
      - PORT=3000
      - DATA_DIR=/data
      - TRUST_PROXY=1
    env_file: .env
    volumes:
      - /mnt/tank/configs/opengym/data:/data
      - /mnt/tank/configs/opengym/coach-auth:/coach-auth
    restart: unless-stopped

    # AI Coach (optional): the default image ships no AI runtime. To opt in, build
    # the coach target instead of pulling — uncomment and run:
    #   API_TARGET=coach docker compose up --build opengym-api
    # build:
    #   context: ./api
    #   target: ${API_TARGET:-default}

  # ── Web: React frontend + nginx, proxies /api; exercise media ship in the image ──
  opengym-web:
    image: ghcr.io/duartesantos8/opengym-web:latest
    container_name: opengym-web
    # No `user:` here on purpose — stock nginx binds :80 and writes /var/cache/nginx
    # and its pid file as root. Forcing 568:568 makes the container crash-loop.
    environment:
      - TZ=America/New_York
      - NGINX_PORT=80
      - BACKEND=opengym-api   # must match the api container/service name above
      - PORT=3000
      # Behind a Cloudflare Tunnel, trust CF's real-client-IP header for the activity log.
      # Leave EMPTY if you ever expose this directly, or clients can forge the address.
      - CF_CONNECTING_IP=$$http_cf_connecting_ip
    ports:
      - "8080:80"
    depends_on:
      opengym-api:
        condition: service_started
    restart: unless-stopped

```

> The exercise images and animations now ship inside the web image, so the old one-shot `opengym-media` job and its `img`/`gif` bind mounts are no longer needed. If you deployed an earlier version of this guide, remove them (mounting over those folders hides the bundled media). Images also moved from `registry.gitlab.com` to `ghcr.io`; the GitLab images are still published.
{.is-info}

1. Create the `.env` file next to the compose file **before** the first start — see section 2.1. Set `RP_ID` and `ORIGIN` to your real domain **before** anyone registers (section 3 explains why)
1. Browse to `http://<host>:8080` and tap **Create profile**





# 2 · Configuration

## 2.1 The `.env` file

The compose file reads `.env` with `env_file`, so `docker compose up` fails if it doesn't exist. Create it in the stack folder:

```env
RP_ID=gym.example.com
ORIGIN=https://gym.example.com
WEB_PORT=8080
RP_NAME=openGym
FIRST_USER_ADMIN=1
```

For a quick local test use `RP_ID=localhost` and `ORIGIN=http://localhost:8080`. With `FIRST_USER_ADMIN=1`, the first profile created on a fresh instance becomes its admin, so register yours before you share the address. See the upstream [`.env.example`](https://github.com/DuarteSantos8/openGym/blob/main/.env.example) for every option.

## 2.2 First run

1. Open the app and tap **Create profile**
2. Register a passkey when prompted (Face ID / Touch ID / Windows Hello / your password manager)
3. Enter your current body weight and set a goal
4. Build a weekly plan — one routine per weekday, drawn from the exercise library
5. Add to your home screen — iOS: **Share → Add to Home Screen** · Android: **⋮ → Add to home screen**

There's also a **guest mode** that keeps everything in the browser's local storage. It works over plain HTTP on a LAN IP, which makes it a reasonable way to poke at the app before you set up a domain — but that data is stranded in that one browser.

## 2.3 Multiple users and admin

Default behavior is open signup: anyone who can reach the URL creates their own profile with isolated data. To lock that down:

```yaml
      - ADMIN_UIDS=youruserid,anotheradmin
      - INVITE_ONLY=1
```

With `FIRST_USER_ADMIN=1` (section 2.1) your first profile is already an admin. For more admins, or an instance that existed before that setting, find the ID under **Settings → Account → Account ID** (or in `data/db.json` under `users[].id`) and drop it into `ADMIN_UIDS`. An **Admin dashboard** link appears in Settings — user list, workout history, the ability to disable an account, and invite-code generation when `INVITE_ONLY=1` is set. Existing accounts keep working when you flip invite-only on.

Admin access is gated by your passkey and enforced server-side, so there's no second login to manage.

# 3 · Reverse proxy and passkeys

This is the step most people skip and then wonder why their phone won't log in. Browsers enforce two rules on WebAuthn: passkeys bind to an exact hostname, and they only work over HTTPS — with `http://localhost` as the single exception.

Put openGym behind whatever you already run, pointed at port `8080`.



# 4 · Notifications

openGym pushes two kinds of alert even when the app is closed: rest-timer-finished, and a nudge on days you have a workout planned but haven't logged one.

Nothing to configure server-side — VAPID keys generate on first run into `data/vapid.json`. Enable it per-profile under **Settings → Notifications**.

Each browser reports its own timezone, so the reminder fires at the user's local time regardless of where the server sits or where they're travelling.

> 
> Notifications require a signed-in passkey profile over HTTPS. Guest mode and plain-HTTP LAN instances can't subscribe, and the option simply won't appear in Settings. Same requirement applies to **Keep screen awake** — the Wake Lock API is HTTPS-only.
{.is-info}


# 5 · Optional: MCP server for AI queries

As of v1.2.5 the repo ships an opt-in **MCP server** that lets Claude Desktop, Cursor, or any MCP client read your training data directly off the `data` folder. It runs as a local stdio process and adds no container and no network exposure.

```bash
git clone https://github.com/DuarteSantos8/openGym
cd openGym/mcp
npm install
```

Claude Desktop config:

```json
{
  "mcpServers": {
    "opengym": {
      "command": "node",
      "args": ["/absolute/path/to/openGym/mcp/src/index.js"],
      "env": {
        "OPENGYM_DATA": "/mnt/tank/configs/opengym/data",
        "OPENGYM_UID": "your-uid"
      }
    }
  }
}
```

`OPENGYM_UID` is auto-detected on a single-profile instance. Eight tools ship in v1 — routines, week plan, workouts, session detail, body weight, estimated 1RM, and muscle balance. The numbers match the Stats screen because the server calls the same functions the UI does.

*"What did I bench last week?"* · *"What's my estimated 1RM on deadlift?"* · *"Which muscles have I been neglecting?"*

> 
> The MCP server never reads passkey material, VAPID keys, or session state — only the profile it's pointed at. Its read tools need no authentication. There is now one write tool, `log_workout`, which stays off until you pair the MCP server with your openGym server — see the [MCP README](https://github.com/DuarteSantos8/openGym/blob/main/mcp/README.md).
{.is-info}

