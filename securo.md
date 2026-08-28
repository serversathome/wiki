---
title: Securo
description: A guide to deploying Securo
published: true
date: 2026-08-28T20:39:44.910Z
tags: 
editor: markdown
dateCreated: 2026-08-28T20:39:44.910Z
---

# <img src="/securo.png" class="tab-icon"> What is Securo?

**Securo** is an open-source, self-hosted personal finance manager — think Rocket Money or Monarch, but running on your own hardware with none of your financial data leaving the house. It handles multi-account tracking with running balances, transactions with search and CSV export, budgets, recurring transactions, savings goals, asset valuation, and Net Worth / Income vs Expenses reporting.


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Securo


```bash
mkdir -p /mnt/tank/configs/securo/{postgres,attachments,secrets,agent_knowledge,embedding_models}
```
```yaml
name: securo

x-backend-env: &backend-env
  DATABASE_URL: postgresql+asyncpg://postgres:postgres@db:5432/securo
  SECRET_KEY: ${SECRET_KEY}
  DEBUG: "false"
  FRONTEND_URL: ${FRONTEND_URL:-http://localhost:3000}
  SIMPLEFIN_ENABLED: ${SIMPLEFIN_ENABLED:-false}
  SIMPLEFIN_API_URL: ${SIMPLEFIN_API_URL:-https://beta-bridge.simplefin.org}
  ENABLE_BANKING_APP_ID: ${ENABLE_BANKING_APP_ID:-}
  ENABLE_BANKING_PRIVATE_KEY_FILE: ${ENABLE_BANKING_PRIVATE_KEY_FILE:-/app/secrets/enable_banking_private.pem}
  ENABLE_BANKING_OAUTH_REDIRECT_URI: ${ENABLE_BANKING_OAUTH_REDIRECT_URI:-}
  PLUGGY_CLIENT_ID: ${PLUGGY_CLIENT_ID:-}
  PLUGGY_CLIENT_SECRET: ${PLUGGY_CLIENT_SECRET:-}
  OPENEXCHANGERATES_APP_ID: ${OPENEXCHANGERATES_APP_ID:-}
  TESOURO_DIRETO_ENABLED: ${TESOURO_DIRETO_ENABLED:-false}
  REDIS_URL: redis://redis:6379/0
  STORAGE_LOCAL_PATH: /app/data/attachments
  AGENTS_KNOWLEDGE_STORAGE_PATH: /app/data/agent_knowledge
  AGENTS_ENABLED: ${AGENTS_ENABLED:-false}
  AGENTS_BUILTIN_MCP_URL: http://mcp-server:8765/mcp
  AGENTS_MCP_JWT_SECRET: ${AGENTS_MCP_JWT_SECRET:-change-me-too}
  AGENTS_DEFAULT_PROVIDER: ${AGENTS_DEFAULT_PROVIDER:-ollama}
  AGENTS_OLLAMA_BASE_URL: ${AGENTS_OLLAMA_BASE_URL:-http://ollama:11434}
  AGENTS_EMBEDDING_PROVIDER: ${AGENTS_EMBEDDING_PROVIDER:-native}

x-backend-base: &backend-base
  image: ghcr.io/securo-finance/securo-backend:latest
  environment: *backend-env
  extra_hosts:
    - "host.docker.internal:host-gateway"
  depends_on:
    db: { condition: service_healthy }
    redis: { condition: service_healthy }
  volumes:
    - /mnt/tank/configs/securo/secrets:/app/secrets:ro
    - /mnt/tank/configs/securo/attachments:/app/data/attachments
    - /mnt/tank/configs/securo/agent_knowledge:/app/data/agent_knowledge
    - /mnt/tank/configs/securo/embedding_models:/app/data/embedding_models
  restart: unless-stopped

services:
  redis:
    image: redis:8-alpine
    container_name: securo-redis
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5

  db:
    image: pgvector/pgvector:pg16
    container_name: securo-db
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: securo
    volumes:
      - /mnt/tank/configs/securo/postgres:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5

  backend:
    <<: *backend-base
    container_name: securo-backend
    command: sh -c "alembic upgrade head && uvicorn app.main:app --host 0.0.0.0 --port 8000"
    ports:
      - "${BACKEND_PORT:-8000}:8000"

  frontend:
    image: ghcr.io/securo-finance/securo-frontend:latest
    container_name: securo-frontend
    ports:
      - "${FRONTEND_PORT:-3000}:8080"
    environment:
      BACKEND_URL: http://backend:8000
      FRONTEND_URL: ${FRONTEND_URL:-http://localhost:3000}
    depends_on:
      - backend
    restart: unless-stopped

  celery-worker:
    <<: *backend-base
    container_name: securo-worker
    command: celery -A app.worker worker --loglevel=info --concurrency=2

  celery-beat:
    <<: *backend-base
    container_name: securo-beat
    command: celery -A app.worker beat --loglevel=info

  mcp-server:
    <<: *backend-base
    container_name: securo-mcp
    profiles: [agents]
    command: uvicorn mcp_server.main:app --host 0.0.0.0 --port 8765
    ports:
      - "${AGENTS_MCP_EXTERNAL_HOST_PORT:-0.0.0.0:8765}:8765"
```

## 1.2 .env File
```bash
openssl rand -hex 32
```

Copy the output — it goes into `SECRET_KEY` below. Do not leave this at the default.

Create `.env` alongside your compose file:

```bash
# ---- Required ----
SECRET_KEY=paste-your-openssl-output-here
FRONTEND_URL=http://192.168.1.50:3000

# ---- Ports ----
FRONTEND_PORT=3000
BACKEND_PORT=8000

# ---- Bank sync: SimpleFIN (US / international) ----
SIMPLEFIN_ENABLED=true
SIMPLEFIN_API_URL=https://beta-bridge.simplefin.org

# ---- Currency conversion (optional) ----
OPENEXCHANGERATES_APP_ID=

# ---- Brazilian Treasury lookup — turn off if you're not in Brazil ----
TESOURO_DIRETO_ENABLED=false
```



> Registration is open by default. Create your admin account immediately after first boot, then go to **Settings → Admin** and disable public registration before you expose this anywhere.
{.is-danger}

# 2 · Bank Sync

Each provider auto-registers as soon as its credentials are present in `.env`. You can run more than one at a time.

# {.tabset}
## SimpleFIN (US)

This is the one most US homelabbers want. SimpleFIN is a **read-only** open protocol — there's no API key to request and no write access to your accounts, ever.

1. Confirm `SIMPLEFIN_ENABLED=true` is in your `.env`
2. Go to [bridge.simplefin.org](https://bridge.simplefin.org/) and link your bank
3. Generate a **Setup Token** (single use)
4. In Securo: **Accounts → Connect Bank → SimpleFIN**
5. Paste the token

For real banks, point `SIMPLEFIN_API_URL` at `https://bridge.simplefin.org`. The default `beta-bridge.simplefin.org` is the sandbox — the [developer page](https://beta-bridge.simplefin.org/info/developers) hands out free demo tokens if you want to test the flow before committing.

> SimpleFIN Bridge is a paid service (a few dollars a year) once you move past the sandbox. Worth knowing before you plan around it.
{.is-info}

## Enable Banking (EU)

Covers roughly 2,500 European PSD2 banks.

1. Sign up at [enablebanking.com](https://enablebanking.com) and create a **Production** application
2. Download the PEM private key and save it to `/mnt/tank/configs/securo/secrets/enable_banking_private.pem`
3. Add to `.env`:

```bash
ENABLE_BANKING_APP_ID=your-application-id
ENABLE_BANKING_PRIVATE_KEY_FILE=/app/secrets/enable_banking_private.pem
ENABLE_BANKING_OAUTH_REDIRECT_URI=https://securo.yourdomain.com/oauth/callback
```

The redirect URI has to match one of the Allowed Redirect URLs in your EB application **exactly**. Enable Banking production requires HTTPS, so this one needs a real domain behind your reverse proxy — a LAN IP will not work.

> On Enable Banking's free tier you must pre-link the accounts you want inside the EB portal *before* connecting from Securo. Skip that and the connection returns zero accounts.
{.is-warning}

## Pluggy (Brazil)

1. Sign up at [pluggy.ai](https://pluggy.ai) and grab your credentials
2. Add `http://<your-host>:3000/oauth/callback` as an allowed redirect URI
3. Add to `.env`:

```bash
PLUGGY_CLIENT_ID=your-client-id
PLUGGY_CLIENT_SECRET=your-client-secret
```

# 3 · Optional Features

## 3.1 Exchange Rates

For automatic multi-currency conversion, grab a free key from [Open Exchange Rates](https://openexchangerates.org/) and add it:

```bash
OPENEXCHANGERATES_APP_ID=your-app-id
```

Rates are fetched on demand when a foreign-currency transaction is created. Without a key, cross-currency amounts fall back to a 1:1 rate with a warning in the UI — usable, but the numbers will be wrong.

## 3.2 AI Agents & MCP

Securo can run self-hosted AI assistants with tool-use over your own financial data, plus a per-agent RAG knowledge base. It supports Ollama, OpenAI, Anthropic, and any OpenAI-compatible endpoint. It's off by default and costs nothing when off.

```bash
AGENTS_ENABLED=true
COMPOSE_PROFILES=agents
```

Redeploy the stack, then go to **Settings → AI Agents** to add a provider connection.

The `agents` profile also starts a built-in **MCP server** on port 8765. From **Settings → AI Agents → External MCP access** you can mint a JWT (90-day default TTL) and point Claude Desktop, n8n, or any other MCP client straight at your own finance data. JWT auth is always required.



## 3.3 File Import

If you're migrating off another finance app, Securo imports **OFX, QIF, CAMT, and CSV**. Pair that with the auto-categorization rules engine and a few years of exported history lands in decent shape without manual tagging.

# 4 · Reverse Proxy & Passkeys

Securo supports TOTP two-factor with brute-force protection, passkeys (WebAuthn), and OIDC login against Authentik, Pocket ID, or any standard provider.

Two WebAuthn rules come from the standard itself, not from Securo:

> **Passkey requirements**
> - [x] Never works on a bare IP address (`http://192.168.1.50:3000`)
> - [x] Never works over plain HTTP, except on `localhost`

So if you want passkeys, put it behind a Cloudflare Tunnel or your reverse proxy of choice and set:

```bash
FRONTEND_URL=https://securo.yourdomain.com
WEBAUTHN_RP_ID=securo.yourdomain.com
```

`FRONTEND_URL` also drives CORS and OAuth callbacks, so it needs to be correct regardless of whether you use passkeys.

For OIDC, register `https://securo.yourdomain.com/api/auth/oidc/callback` as the callback in your provider, then:

```bash
OIDC_ENABLED=true
OIDC_PROVIDER_NAME=Pocket ID
OIDC_DISCOVERY_URL=https://id.yourdomain.com/.well-known/openid-configuration
OIDC_CLIENT_ID=securo
OIDC_CLIENT_SECRET=your-client-secret
# Optional: OIDC-only, kills local login and registration entirely
LOCAL_AUTH_ENABLED=false
```

