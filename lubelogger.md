---
title: Lubelogger
description: A guide to deploying Lubelogger
published: true
date: 2026-02-10T11:01:46.468Z
tags: 
editor: markdown
dateCreated: 2026-02-10T11:00:58.064Z
---

# <img src="/lubelogger.png" class="tab-icon"> What is LubeLogger?

**LubeLogger** is a self-hosted, open-source, web-based vehicle maintenance and fuel mileage tracker. Despite its unconventional name, it's an incredibly powerful tool for keeping all your vehicle service records, fuel economy data, repair history, and maintenance reminders organized in one place. Whether you're managing a single daily driver or a fleet of vehicles, LubeLogger gives you a clean dashboard with expense breakdowns, fuel economy charts, and upcoming maintenance reminders.



# 1 · Deploy LubeLogger
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

### Basic Deployment (SQLite)

This is the simplest deployment using LubeLogger's built-in SQLite file-based storage.

```yaml
services:
  lubelogger:
    image: ghcr.io/hargata/lubelogger:latest
    container_name: lubelogger
    restart: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - /mnt/tank/configs/lubelogger/data:/App/data
      - /mnt/tank/configs/lubelogger/keys:/root/.aspnet/DataProtection-Keys
    environment:
      - LC_ALL=en_US.UTF-8
      - LANG=en_US.UTF-8
```

### Production Deployment (PostgreSQL)

For a more robust setup, use PostgreSQL as the database backend. Create a `.env` file alongside your compose file for environment variables.

```yaml
services:
  lubelogger:
    image: ghcr.io/hargata/lubelogger:latest
    container_name: lubelogger
    restart: unless-stopped
    depends_on:
      lubelogger-db:
        condition: service_healthy
    ports:
      - "8080:8080"
    volumes:
      - /mnt/tank/configs/lubelogger/data:/App/data
      - /mnt/tank/configs/lubelogger/keys:/root/.aspnet/DataProtection-Keys
    env_file:
      - .env
    environment:
      - LC_ALL=en_US.UTF-8
      - LANG=en_US.UTF-8
      - POSTGRES_CONNECTION=Host=lubelogger-db;Port=5432;Username=lubelogger;Password=CHANGE_ME;Database=lubelogger;

  lubelogger-db:
    image: postgres:16
    container_name: lubelogger-db
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "pg_isready", "-q", "-d", "lubelogger", "-U", "lubelogger"]
      timeout: 45s
      interval: 10s
      retries: 10
    environment:
      - POSTGRES_USER=lubelogger
      - POSTGRES_PASSWORD=CHANGE_ME
      - POSTGRES_DB=lubelogger
    volumes:
      - /mnt/tank/configs/lubelogger/db:/var/lib/postgresql/data
```

> 
> Make sure to change `CHANGE_ME` to a strong, unique password in both the `POSTGRES_CONNECTION` string and the `POSTGRES_PASSWORD` field. These must match.
{.is-warning}

> 
> If using PostgreSQL, ensure the `app` schema exists in the database. LubeLogger requires this schema to operate. If you encounter errors on first launch, you may need to create it manually with: `CREATE SCHEMA IF NOT EXISTS app;`
{.is-info}

## <img src="/truenas.png" class="tab-icon"> TrueNAS

LubeLogger is available in the **Community** train of the TrueNAS App Catalog.

1. Navigate to **Apps** in the TrueNAS UI
2. Click **Discover Apps**
3. Search for **LubeLogger**
4. Click **Install**
5. Configure the following settings:
   - **Network → Web Port**: `8080` (or your preferred port)
   - **Storage → App Data Host Path**: `/mnt/tank/configs/lubelogger/data`
   - **Storage → Keys Host Path**: `/mnt/tank/configs/lubelogger/keys`
6. Click **Install**

> 
> Make sure the **Community** train is enabled in your TrueNAS App catalog settings. Go to **Apps → Discover Apps → Settings** and check that the **Community** train is selected under **Preferred Trains**.
{.is-info}

# 2 · Configuration

## 2.1 Initial Setup

1. Open your browser and navigate to `http://YOUR-IP:8080`
2. Click the **Settings** icon (gear) in the top right corner
3. Recommended first-time settings:
   - **Enable Dark Mode** if preferred
   - **Enable Authentication** — this will prompt you to create a root username and password
   - Configure your **locale** settings for proper currency and date formatting

> 
> Authentication is **not enabled by default**. Anyone with access to the URL can view and modify your data until you enable it. Enable authentication immediately after deployment.
{.is-danger}

## 2.2 Server Settings Configurator

LubeLogger includes a built-in **Server Settings Configurator** accessible at `/setup` or from the **Settings** tab. This wizard walks you through configuring:

- **Locale & Language** — affects number, currency, and date formatting
- **Email / SMTP** — required for multi-user registration tokens and password resets
- **PostgreSQL Connection** — if migrating from SQLite to Postgres
- **HTTPS / Kestrel** — for configuring SSL certificates
- **OpenID Connect** — for SSO integration

> 
> You can also use the online [LubeLogger Configurator](https://lubelogger.com/configure/) to generate your `.env` file with all the correct environment variables before deployment.
{.is-success}

## 2.3 Adding Vehicles

1. Navigate to the **Garage** tab
2. Click the green **+** button to add a new vehicle
3. Enter the vehicle's make, model, year, license plate, and other details
4. Optionally upload a photo for the vehicle
5. Click on a vehicle to access its full record tabs: Service, Repairs, Upgrades, Fuel, Taxes, Odometer, Reminders, Notes, and more

## 2.4 Environment Variables

LubeLogger supports a wide range of environment variables for advanced configuration. Some commonly used ones:

| Variable | Description |
|---|---|
| `LC_ALL` / `LANG` | Locale and language settings (e.g., `en_US.UTF-8`) |
| `POSTGRES_CONNECTION` | PostgreSQL connection string |
| `MailConfig__EmailServer` | SMTP server for email notifications |
| `MailConfig__Port` | SMTP port (use `587` for TLS) |
| `MailConfig__Username` | SMTP username |
| `MailConfig__Password` | SMTP password |
| `LUBELOGGER_MOTD` | Message of the Day on login page |
| `LUBELOGGER_WEBHOOK` | Webhook URL for notifications |
| `LOGGING__LOGLEVEL__DEFAULT` | Log level (`Error`, `Warning`, etc.) |


> 
> For a full list of environment variables, see the [official documentation](https://docs.lubelogger.com/Advanced/Environment%20Variables).
{.is-info}

## 2.5 Backups

LubeLogger has a built-in backup system accessible from the **Settings** tab. Log in as the root user and click **Create Backup** to download a zip file containing all your data. You can also automate backups via the API using a cron job.
