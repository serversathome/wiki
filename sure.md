---
title: Sure
description: A guide to deploy Sure
published: true
date: 2026-08-06T14:22:23.744Z
tags: 
editor: markdown
dateCreated: 2026-08-06T14:22:23.744Z
---

# <img src="/sure.png" class="tab-icon"> What is Sure?

**Sure** is a self-hosted personal finance app — net worth tracking, account aggregation, spending breakdowns, and investment holdings. It's the community fork of Maybe Finance, the VC-backed personal finance startup that shut down in 2023 and open-sourced its codebase.



# 1 · Deploy Sure
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

```yaml
x-db-env: &db_env
  POSTGRES_USER: ${POSTGRES_USER:-sure_user}
  POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:?set POSTGRES_PASSWORD in .env}
  POSTGRES_DB: ${POSTGRES_DB:-sure_production}

x-rails-env: &rails_env
  <<: *db_env
  SECRET_KEY_BASE: ${SECRET_KEY_BASE:?generate with openssl rand -hex 64}
  SELF_HOSTED: "true"
  RAILS_FORCE_SSL: ${RAILS_FORCE_SSL:-false}
  RAILS_ASSUME_SSL: ${RAILS_ASSUME_SSL:-false}
  DB_HOST: db
  DB_PORT: 5432
  REDIS_URL: redis://redis:6379/1
  # Optional — leave blank to disable AI features entirely
  OPENAI_ACCESS_TOKEN: ${OPENAI_ACCESS_TOKEN:-}
  OPENAI_URI_BASE: ${OPENAI_URI_BASE:-}
  OPENAI_MODEL: ${OPENAI_MODEL:-}
  OPENAI_REQUEST_TIMEOUT: ${OPENAI_REQUEST_TIMEOUT:-60}
  OPENAI_SUPPORTS_PDF_PROCESSING: ${OPENAI_SUPPORTS_PDF_PROCESSING:-true}

services:
  web:
    image: ghcr.io/we-promise/sure:stable
    container_name: sure-web
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/sure/storage:/rails/storage
    ports:
      - ${PORT:-3000}:3000
    environment:
      <<: *rails_env
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy

  worker:
    image: ghcr.io/we-promise/sure:stable
    container_name: sure-worker
    command: bundle exec sidekiq
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/sure/storage:/rails/storage
    environment:
      <<: *rails_env
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy

  db:
    image: postgres:16
    container_name: sure-db
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/sure/postgres:/var/lib/postgresql/data
    environment:
      <<: *db_env
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $$POSTGRES_USER -d $$POSTGRES_DB"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: sure-redis
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/sure/redis:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5
```

Create the data directories and set ownership **before** first start:

```bash
mkdir -p /mnt/tank/configs/sure/{postgres,redis,storage}
chown -R 999:999   /mnt/tank/configs/sure/postgres
chown -R 999:1000  /mnt/tank/configs/sure/redis
chown -R 1000:1000 /mnt/tank/configs/sure/storage
```

> **Ownership is not uniform.** Postgres and Redis both run as uid 999 but with different gids, and the Rails app runs as 1000. Using 568 across the board — the usual Servers@Home convention — will fail on all three.
{.is-danger}

Then create your `.env` alongside the compose file:

```bash
SECRET_KEY_BASE=          # openssl rand -hex 64
POSTGRES_PASSWORD=
POSTGRES_USER=sure_user
POSTGRES_DB=sure_production
PORT=3000
RAILS_FORCE_SSL=false
RAILS_ASSUME_SSL=false
```

> Leave the Postgres data directory at `0700`. Postgres checks the mode on startup and refuses to initialize if it's group- or world-readable. The image sets this itself on first init — don't `chmod 755` it out of habit.
{.is-warning}

## <img src="/truenas.png" class="tab-icon"> TrueNAS

A community app landed in the TrueNAS catalog in March 2026.

1. Navigate to **Apps** in the TrueNAS UI
2. Search for **Sure** (Community train, Financial category)
3. Click **Install**
4. Configure:
   - **Host Path** for app storage: `/mnt/tank/configs/sure/storage`
   - **Host Path** for Postgres: `/mnt/tank/configs/sure/postgres`
   - **Web Port**: `3000` (or your preference)
   - Generate and paste a **Secret Key Base** — `openssl rand -hex 64`
5. Click **Save**

> The catalog app bundles Postgres and Redis, so you don't manage those separately. The tradeoff is less control over versions and image tags — if you want to pin a specific Sure release, the Docker route is cleaner.
{.is-info}

# 2 · First Login

Browse to `http://<host>:3000` and create your account. The first account registered becomes the admin.

> If you're fronting this with a reverse proxy, set both `RAILS_FORCE_SSL` and `RAILS_ASSUME_SSL` to `true`. Otherwise Rails generates `http://` URLs behind the proxy and login redirects break in a way that's genuinely annoying to diagnose.
{.is-warning}

# 3 · Bank Sync with SimpleFIN

SimpleFIN is an open protocol for financial aggregation. Since almost no US bank runs its own SimpleFIN server, in practice you'll use **SimpleFIN Bridge**, which acts as the aggregator. It runs about $15/year.

## 3.1 Set Up SimpleFIN Bridge

1. Sign up at [bridge.simplefin.org](https://bridge.simplefin.org) and pay — it won't issue tokens otherwise
2. Under **Financial Institutions**, click **New connection** and link each bank and credit card
3. Confirm every institution shows status **OK**
4. Enable **Passkeys** or **2FA** on the Bridge account while you're there

> Link *all* your institutions before generating a token. The token grants access to whatever is connected at claim time, and adding banks afterward means going through this again.
{.is-info}

## 3.2 Generate the Setup Token

1. Under **Apps**, click **New app connection**
2. Select which accounts to share
3. Copy the setup token — a long base64 string. Grab all of it, with no trailing whitespace or line breaks

## 3.3 Connect Sure

In Sure, go to **Settings → SimpleFIN Connections** and paste the token. Sure exchanges it for a long-lived access URL containing HTTP Basic Auth credentials, and stores that encrypted with your `SECRET_KEY_BASE`.


> **Never manually retry `SimplefinConnectionUpdateJob`.** It consumes a single-use setup token, and retrying permanently breaks that connection attempt. If a sync fails, go back to Bridge, generate a *fresh* token, and start over.
{.is-danger}

## 3.4 What to Expect

| Behavior | Notes |
|----------|-------|
| Refresh cadence | Roughly every 24 hours — not real time |
| Transaction lag | New transactions often take a few days to appear |
| Credit card sign | Some institutions report positive, others negative — reconcile manually |
| Pending transactions | Not all institutions return them, even when requested |
| Non-USD accounts | Known bug where these can import as USD — spot-check |


> Rotating `SECRET_KEY_BASE` invalidates the stored access URL. You'll need to claim a brand new token from Bridge. Note this wherever you keep your secrets.
{.is-warning}

# 4 · AI Features (Optional)

Sure can use an LLM for chat, auto-categorization, merchant detection, and PDF statement parsing. It works with **any** OpenAI-compatible endpoint.

| Provider | Configuration |
|----------|---------------|
| OpenAI | Set `OPENAI_ACCESS_TOKEN` only |
| OpenRouter | `OPENAI_URI_BASE=https://openrouter.ai/api/v1` + model name |
| Ollama / LM Studio | Point `OPENAI_URI_BASE` at the `/v1` path, use any dummy token |


Notes worth having:

- The `/v1` suffix is required. Omitting it produces an unhelpful 404
- If the endpoint has no vision support, set `OPENAI_SUPPORTS_PDF_PROCESSING=false`


# <img src="/youtube.png" class="tab-icon"> 5 · Video

