---
title: CompDesk
description: A guide to deploying CompDesk
published: true
date: 2026-09-28T18:10:28.190Z
tags: 
editor: markdown
dateCreated: 2026-09-28T18:10:28.190Z
---

# What is CompDesk?

**CompDesk** is a lightweight, privacy-first help desk and ticketing system for small organizations. Tickets route to departments, and four server-enforced roles (User, Agent, department Admin and Super Admin) control who sees what. It includes searchable queues, templates, custom fields, SLA policies and escalation, internal notes, private attachments with optional ClamAV scanning, SMTP notifications, signed webhooks, your own branding, and optional Microsoft Entra ID sign-in, with English and French interfaces.

<div class="glance">
  <div><span>Port</span><b><code>3070</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>3 services</b></div>
  <div><span>Depends on</span><b>Postgres</b></div>
  <div><span>Difficulty</span><b class="difficulty intermediate">Intermediate</b></div>
  <div class="glance-links">
    <a href="https://github.com/TahaHydra/CompDesk"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>


# <img src="/docker.png" class="tab-icon"> 1 · Deploy CompDesk

The stack is three services that share one image:

| Service | Job |
|---------|-----|
| `config-init` | Runs once, fixes folder ownership and generates the Postgres password into the config folder |
| `db` | Postgres 16, which reads its password from that file |
| `compdesk` | Serves the setup wizard, then switches itself to the production app on the same port |


```bash
mkdir -p /mnt/tank/configs/compdesk/{config,postgres,uploads,attachments}
chown -R 568:568 /mnt/tank/configs/compdesk
```

```yaml
services:
  config-init:
    image: ghcr.io/tahahydra/compdesk:0.9.0-beta.2
    container_name: compdesk-init
    user: root
    restart: "no"
    security_opt:
      - no-new-privileges:true
    command: ["node", "scripts/config-init.mjs"]
    environment:
      COMPDESK_CONFIG_DIR: /config
      COMPDESK_PGDATA_CHECK_DIR: /pgdata-check
      COMPDESK_RUNTIME_UID: "568"
      COMPDESK_RUNTIME_GID: "568"
      COMPDESK_CHOWN_PATHS: "/config:/app/public/uploads:/app/storage/attachments"
    volumes:
      - /mnt/tank/configs/compdesk/config:/config
      - /mnt/tank/configs/compdesk/postgres:/pgdata-check:ro
      - /mnt/tank/configs/compdesk/uploads:/app/public/uploads
      - /mnt/tank/configs/compdesk/attachments:/app/storage/attachments

  db:
    image: postgres:16-alpine
    container_name: compdesk-db
    restart: on-failure:5
    security_opt:
      - no-new-privileges:true
    read_only: true
    tmpfs:
      - /tmp
      - /var/run/postgresql
    environment:
      POSTGRES_USER: compdesk
      POSTGRES_DB: compdesk
      POSTGRES_PASSWORD_FILE: /run/compdesk-config/secrets/postgres_password
    depends_on:
      config-init:
        condition: service_completed_successfully
    volumes:
      - /mnt/tank/configs/compdesk/postgres:/var/lib/postgresql/data
      - /mnt/tank/configs/compdesk/config:/run/compdesk-config:ro
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U compdesk -d compdesk"]
      interval: 5s
      timeout: 5s
      retries: 30
      start_period: 60s
    stop_grace_period: 30s

  compdesk:
    image: ghcr.io/tahahydra/compdesk:0.9.0-beta.2
    container_name: compdesk
    user: "568:568"
    restart: on-failure:5
    init: true
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    read_only: true
    tmpfs:
      - /tmp
    depends_on:
      config-init:
        condition: service_completed_successfully
      db:
        condition: service_healthy
    environment:
      COMPDESK_CONFIG_DIR: /config
      COMPDESK_DB_HOST: db
      COMPDESK_DB_PORT: "5432"
      COMPDESK_PUBLISHED_PORT: "3070"
      SETUP_PUBLIC_ORIGIN: https://helpdesk.example.com
      SETUP_TRUST_PROXY: "true"
    ports:
      - "3070:3000"
    volumes:
      - /mnt/tank/configs/compdesk/config:/config
      - /mnt/tank/configs/compdesk/uploads:/app/public/uploads
      - /mnt/tank/configs/compdesk/attachments:/app/storage/attachments
    healthcheck:
      test: ["CMD", "node", "-e", "fetch('http://127.0.0.1:3000/api/health/live').then(r=>{if(!r.ok)process.exit(1)}).catch(()=>process.exit(1))"]
      interval: 15s
      timeout: 5s
      retries: 5
      start_period: 120s
    stop_grace_period: 40s
```

1. Point a Cloudflare Tunnel (or other HTTPS reverse proxy) hostname at `http://your-server-ip:3070`
2. Set `SETUP_PUBLIC_ORIGIN` to that exact HTTPS origin
3. Deploy the stack

> This guide pins `0.9.0-beta.2` instead of `latest` on purpose. CompDesk only publishes a `latest` tag for stable releases, and every release so far is a beta, so `latest` does not exist yet. Check the GitHub releases page for newer versions when you update.
{.is-warning}

> CompDesk must be served over **HTTPS** anywhere other than `localhost`. That is why this compose sets `SETUP_PUBLIC_ORIGIN` and `SETUP_TRUST_PROXY`. Only enable `SETUP_TRUST_PROXY` when the proxy or tunnel is the only way in and it overwrites the forwarded host and protocol headers.
{.is-danger}

> `config-init` and `db` keep their upstream users. `config-init` has to start as root to chown the folders (to `568:568`, via `COMPDESK_RUNTIME_UID`/`GID`), and the Postgres entrypoint manages its own data folder ownership. The `compdesk` app container itself runs as the apps user.
{.is-info}

# 2 · Configuration

## 2.1 First-Run Setup Wizard

1. Grab the one-time setup token from the logs. It is printed in a clearly boxed section and expires after 30 minutes:

2. Browse to `https://helpdesk.example.com/setup` and paste in the token
3. Choose the bundled Postgres option. The wizard tests the database, writes its secrets, runs migrations and creates the first **Super Admin**
4. When asked, select **Trust a configured reverse proxy** so the running app reads client IPs and HTTPS from your tunnel correctly
5. Finish the wizard. The same container switches to the production app automatically, and setup is permanently disabled from then on

> If the token expires, restart the `compdesk` container and a new one is printed.
{.is-info}

## 2.2 Departments and Roles

Create departments first, then invite agents into them. Tickets are scoped to a department, and the roles are enforced on the server:

| Role | Can do |
|------|--------|
| User | Open and follow their own tickets |
| Agent | Work tickets in their departments |
| Admin | Manage one department's settings and agents |
| Super Admin | Everything, including branding, SMTP, API clients and webhooks |


## 2.3 Email, Branding and Integrations

As a Super Admin, set up SMTP for notifications, upload your logo and colors, and add signed webhooks or department-scoped API clients if you need them. The external API is off by default on new installs.


