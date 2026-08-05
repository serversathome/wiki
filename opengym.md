---
title: openGym
description: A guide to deploy openGym
published: true
date: 2026-08-05T20:35:06.941Z
tags: 
editor: markdown
dateCreated: 2026-08-05T20:35:06.941Z
---

# What is openGym?

**openGym** is a self-hosted gym and body-weight tracker — a fully private replacement for Strong, Hevy, or the fitness app that came with your phone. You plan a routine for each day of the week, run guided workouts that pre-fill your weights from last session, and track body weight against a goal line. It ships with a library of **1,324 exercises** complete with animated demos, supports supersets and cardio logging, detects PRs, and draws a GitHub-style activity heatmap of your training year.

It installs to your home screen as a PWA, signs you in with **passkeys** (Face ID / Touch ID / fingerprint) instead of passwords, and syncs across your phone and laptop. There is no database server, no cloud dependency, and **no telemetry** — everything lives as plain JSON files in a folder you control.

Stack is React 19 + Vite on the front, a dependency-light Node API on the back, nginx in front of both. Licensed AGPL v3.


# <img src="/docker.png" class="tab-icon"> 1 · Deploy openGym


```yaml
services:
  # One-time job: pulls the exercise images + GIFs (~140 MB) into the media folder.
  # Exits when done, and skips the download on every start after the first.
  media:
    image: alpine/git
    volumes:
      - /mnt/tank/configs/opengym/media/img:/out/img
      - /mnt/tank/configs/opengym/media/gif:/out/gif
    entrypoint: ["/bin/sh", "-c"]
    command:
      - |
        if [ -z "$$(ls -A /out/img 2>/dev/null)" ]; then
          echo "Downloading exercise media (~140 MB, one time)..."
          git clone --depth 1 https://github.com/hasaneyldrm/exercises-dataset /tmp/ds
          cp /tmp/ds/images/*.jpg /out/img/ && cp /tmp/ds/videos/*.gif /out/gif/
          echo "Exercise media ready."
        else
          echo "Exercise media already present - skipping download."
        fi
    restart: "no"

  api:
    image: ghcr.io/duartesantos8/opengym-api:latest
    container_name: opengym-api
    restart: unless-stopped
    environment:
      - PORT=3000                        # internal only - nginx proxies here, do not change
      - DATA_DIR=/data
      - RP_ID=localhost                  # hostname passkeys bind to
      - ORIGIN=http://localhost:8080     # exact URL in the address bar
      - RP_NAME=openGym                  # name shown in the passkey prompt
      - SESSION_DAYS=90
    volumes:
      - /mnt/tank/configs/opengym/data:/data

  web:
    image: ghcr.io/duartesantos8/opengym-web:latest
    container_name: opengym-web
    restart: unless-stopped
    depends_on:
      media:
        condition: service_completed_successfully
      api:
        condition: service_started
    ports:
      - "8080:80"
    volumes:
      - /mnt/tank/configs/opengym/media/img:/usr/share/nginx/html/img:ro
      - /mnt/tank/configs/opengym/media/gif:/usr/share/nginx/html/gif:ro
```

1. Set `RP_ID` and `ORIGIN` to your real domain **before** anyone registers — see section 3
1. Browse to `http://<host>:8080` and tap **Create profile**





# 2 · Configuration


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

Register your own profile first, then find your ID in `data/db.json` under `users[].id` and drop it into `ADMIN_UIDS`. An **Admin dashboard** link appears in Settings — user list, workout history, the ability to disable an account, and invite-code generation when `INVITE_ONLY=1` is set. Existing accounts keep working when you flip invite-only on.

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

As of v1.2.5 the repo ships an opt-in **MCP server** that lets Claude Desktop, Cursor, or any MCP client read your training data directly off the `data` folder. It's read-only, runs as a local stdio process, and adds no container and no network exposure.

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
> The MCP server never reads passkey material, VAPID keys, or session state — only the profile it's pointed at. It also can't write, so you can't log a workout through it yet.
{.is-info}

