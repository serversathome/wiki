---
title: May
description: A guide to deploy May
published: true
date: 2026-03-02T15:20:14.740Z
tags: 
editor: markdown
dateCreated: 2026-03-02T15:20:14.740Z
---

# What is May?

**May** is a modern, self-hosted vehicle management application for tracking fuel consumption, expenses, reminders, and maintenance across your entire fleet. It provides a clean, web-based dashboard for logging fuel fill-ups, monitoring costs, and planning maintenance for all of your vehicles. The application supports multiple users, multiple vehicles per user, and includes features like PDF reports, notifications (via email, ntfy, Pushover, or webhooks), document storage, and Home Assistant integration.

Named after James May, completing the trio of Top Gear presenters alongside [Clarkson](https://github.com/linuxserver/Clarkson) and [Hammond](https://github.com/AlfHou/hammond).



# <img src="/docker.png" class="tab-icon"> 1 · Deploy May

```yaml
services:
  may:
    image: ghcr.io/dannymcc/may:latest
    container_name: may
    restart: unless-stopped
    ports:
      - "5050:5050"
    volumes:
      - /mnt/tank/configs/may:/app/data
    environment:
      - SECRET_KEY=change-me-to-a-secure-random-string
      - DATABASE_URL=sqlite:////app/data/may.db
      - UPLOAD_FOLDER=/app/data/uploads
```


> On first run, if no `ADMIN_PASSWORD` environment variable is set, May generates a secure random password and prints it to the container logs. Set `ADMIN_PASSWORD` in your environment to use a fixed password instead.
{.is-warning}

# 2 · Configuration

## 2.1 Initial Setup

After logging in for the first time, you'll want to configure a few things:

1. Navigate to **Settings** to customize your preferences
2. Set your preferred **units** (metric or imperial) and **currency**
3. Add your first vehicle under **Vehicles** — enter the make, model, year, registration, fuel type, and tank capacity
4. Optionally upload a photo for each vehicle

## 2.2 Adding Fuel Logs

Each time you fill up, record the data in May:

1. Click **Add Fuel Log** or use **Quick Entry Mode** for a streamlined experience
2. Enter the date, odometer reading, fuel amount, total cost, and whether it was a full tank
3. May automatically calculates consumption (L/100km or MPG) based on full-tank fill-ups

## 2.3 Expenses & Maintenance

May supports comprehensive expense tracking and maintenance scheduling:

| Category | Examples |
|----------|----------|
| Maintenance & Repairs | Oil changes, tire rotations, brake pads |
| Insurance | Annual or monthly premiums |
| Tax & Registration | Road tax, registration renewals |
| Parking & Tolls | Parking fees, toll charges |
| Accessories | Dash cams, floor mats, upgrades |

Set up **Maintenance Schedules** with mileage or time-based intervals (e.g., oil change every 10,000 km or 12 months) and May will generate reminders automatically.

## 2.4 Reminders & Notifications

Configure notifications to never miss important dates:

1. Go to **Settings > Notifications**
2. Choose your notification method: **Email**, **ntfy**, **Pushover**, or **Webhook**
3. Set up reminders for MOT/inspections, service intervals, insurance renewals, and tax payments

> 
> Webhook notifications support HTTP POST, making it easy to integrate with Home Assistant, Discord, or Slack.
{.is-info}

## 2.5 Integrations

### Home Assistant

May provides dedicated API endpoints for Home Assistant:

- `/api/ha/status` — System status
- `/api/ha/vehicles` — Vehicle list and stats
- `/api/ha/alerts` — Active alerts and reminders
- `/api/ha/summary` — Overview summary

Generate an API key under **Settings > API** and use it with a REST sensor in Home Assistant.

### Calendar Subscription

Subscribe to your reminders in any calendar app:

1. Go to **Settings > Integrations > Calendar**
2. Copy the **webcal** URL (for Apple Calendar, Outlook) or **HTTPS** URL (for Google Calendar)
3. Add as a subscribed calendar — maintenance schedules, recurring expenses, document expiry dates, and custom reminders will appear automatically

### DVLA Integration (UK)

If you're in the UK, May can look up vehicle MOT and tax status automatically via the DVLA API. Configure your API key under **Admin Settings > DVLA API**.


## 2.6 Import & Export

May supports importing fuel data from **Fuelly CSV** format and exporting all data as **JSON** or **CSV** for backup or migration purposes. Access these options under **Settings > Import/Export**.

> 
> Regularly export your data as a backup. May uses SQLite by default, so you can also back up the database file directly from `/mnt/tank/configs/may/may.db`.
{.is-success}

# <img src="/youtube.png" class="tab-icon"> 3 · Video

*Video coming soon — check back for a full walkthrough!*