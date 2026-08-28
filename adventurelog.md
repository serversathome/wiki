---
title: AdventureLog
description: A guide to deploying AdventureLog
published: true
date: 2026-08-28T20:55:40.725Z
tags: 
editor: markdown
dateCreated: 2026-08-28T20:55:40.725Z
---

# <img src="/adventurelog.png" class="tab-icon"> What is AdventureLog?

**AdventureLog** is a self-hostable travel tracker and trip planner built by Sean Morley. It gives you a private place to log everywhere you've been, plan where you're going next, and see it all on an interactive world map.

Log locations with photos, dates, ratings, tags, and notes. Sort them into custom categories, mark them visited or planned, and drop new pins straight from the map. The world travel book tracks which countries and regions you've checked off, and a dashboard rolls it all up into travel stats.

For trips, the itinerary planner handles any number of days and destinations, with flight info, notes, checklists, and external links. Itineraries can be shared with friends and family so everyone plans together, and locations can be shared publicly or kept private.



# 1 · Deploy AdventureLog
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

```yaml
services:
  app:
    image: ghcr.io/seanmorley15/adventurelog:latest
    container_name: adventurelog
    restart: unless-stopped
    environment:
      SITE_URL: http://192.168.1.100:8015
      POSTGRES_PASSWORD: SuperSecretDBPassword
      PGHOST: db
      POSTGRES_DB: database
      POSTGRES_USER: adventure
      PORT: "3000"
    ports:
      - "8015:80"
    volumes:
      - /mnt/tank/configs/adventurelog/media:/code/media/
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test:
        [
          "CMD-SHELL",
          "python3 -c \"import urllib.request; urllib.request.urlopen('http://127.0.0.1:80/health')\""
        ]
      interval: 30s
      timeout: 5s
      retries: 5
      start_period: 120s

  db:
    image: postgis/postgis:16-3.5
    container_name: adventurelog-db
    restart: unless-stopped
    environment:
      POSTGRES_DB: database
      POSTGRES_USER: adventure
      POSTGRES_PASSWORD: SuperSecretDBPassword
    volumes:
      - /mnt/tank/configs/adventurelog/db:/var/lib/postgresql/data/
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U adventure -d database"]
      interval: 5s
      timeout: 5s
      retries: 5
      start_period: 10s
```

1. Replace `192.168.1.100` with your server's IP or domain
2. Set a real `POSTGRES_PASSWORD` — it must match between the `app` and `db` services
3. Create the host directories first: `mkdir -p /mnt/tank/configs/adventurelog/{db,media}`

> **First boot takes a few minutes.** The `start_period: 120s` on the healthcheck is there for a reason — the backend imports the world geography dataset before it reports healthy. Don't panic if the container shows unhealthy for the first two minutes.
{.is-info}

## <img src="/truenas.png" class="tab-icon"> TrueNAS

AdventureLog is in the TrueNAS **Community** train (added September 2025).

1. Navigate to **Apps** → **Discover Apps** in the TrueNAS UI
2. Search for **AdventureLog** and click **Install**
3. Configure the following:
   - **Web Port**: `8015`
   - **Backend Port**: `8016`
   - **Database Password**: set a strong value
   - **Django Admin Username / Password / Email**: your first-boot superuser
   - **Public URL**: `http://TRUENAS_IP:8016`
   - **Frontend URL**: `http://TRUENAS_IP:8015`
   - **Storage**: point the app and Postgres datasets at host paths under `/mnt/tank/configs/adventurelog`
4. Click **Install** and wait for all three containers to report healthy

> The app deploys three containers — frontend, backend, and PostGIS. The **backend container runs as root (UID/GID 0)**, while the frontend and postgis containers run as UID 1000. Set dataset ownership accordingly or the backend will fail its health check.
{.is-warning}

> First boot can take several minutes while the world geography dataset imports. If the backend keeps reporting unhealthy after a fresh install or a `docker system prune`, remove the app and redeploy — a stale Docker network is a known cause of `failed to set up container networking` on TrueNAS 25.10.x.
{.is-info}

# 2 · First Login

Browse to `http://YOUR_IP:8015` and sign in with the default credentials:

| Field    | Default |
|----------|---------|
| Username | `admin` |
| Password | `admin` |


> Change the admin password immediately after your first login. **Settings → Account**
{.is-danger}

Once you're in, the Django admin panel lives at `http://YOUR_IP:8016/admin` (Standard Deployment: `SITE_URL/admin`). Use it for user management, region data, and anything the main UI doesn't expose.


# 3 · Configuration

## 3.1 Locking Down Registration

Running this for just yourself and your family? Kill open signups:

```bash
DISABLE_REGISTRATION=True
DISABLE_REGISTRATION_MESSAGE=Registration is disabled on this instance.
```

Then invite users manually from the Django admin panel. `ACCOUNT_EMAIL_VERIFICATION` accepts `none`, `optional`, or `mandatory` if you've got SMTP configured.

## 3.2 Rate Limiting

Off by default. If AdventureLog is exposed to the internet, turn it on:

```bash
ENABLE_RATE_LIMITS=True
```

The per-endpoint defaults (`RATE_LIMIT_IMAGE_PROXY=60/minute`, `RATE_LIMIT_EXTERNAL_GEOCODE=120/minute`, and friends) are sensible — the main thing is flipping the master switch.

## 3.3 Reverse Proxy

Behind NPM, Traefik, or Caddy, set `SITE_URL` to your public HTTPS URL and proxy to the container port. On the Advanced Deployment you'll need to expose the **backend** on its own hostname too, then update `PUBLIC_URL`, `FRONTEND_URL`, and `CSRF_TRUSTED_ORIGINS` to match.

> **Images not loading behind a proxy?** That's almost always `PUBLIC_URL` pointing somewhere the browser can't reach. It's used to build image URLs, so it has to be the address *your browser* uses — not an internal one.
{.is-warning}

## 3.4 Integrations

| Integration | Variables |
|-------------|-----------|
| Google Maps | `GOOGLE_MAPS_API_KEY` |
| Strava | `STRAVA_CLIENT_ID`, `STRAVA_CLIENT_SECRET` |
| S3 media storage | `MEDIA_STORAGE=s3`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_STORAGE_BUCKET_NAME`, `AWS_S3_ENDPOINT_URL` |
| SMTP email | `EMAIL_BACKEND=email`, `EMAIL_HOST`, `EMAIL_PORT`, `EMAIL_HOST_USER`, `EMAIL_HOST_PASSWORD`, `DEFAULT_FROM_EMAIL` |
| Umami analytics | `PUBLIC_UMAMI_SRC`, `PUBLIC_UMAMI_WEBSITE_ID` |


Immich, Wanderer, and Endurain are configured **in the app UI** under integration settings rather than through environment variables. The Immich one is the standout — it pulls photos from your existing Immich library straight into your location entries instead of duplicating them.

S3 storage works with Cloudflare R2, MinIO, and DigitalOcean Spaces via `AWS_S3_ENDPOINT_URL`. For R2, set `AWS_S3_REGION_NAME=auto`.
