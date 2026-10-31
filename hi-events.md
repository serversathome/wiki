---
title: Hi.Events
description: A guide to deploying Hi.Events
published: true
date: 2026-10-31T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-10-31T12:00:00.000Z
---

# <img src="/hi-events.png" class="tab-icon"> What is Hi.Events?

**Hi.Events** is a self-hosted ticketing and event management platform, an open source alternative to Eventbrite. Sell tickets with your own branding and checkout, take payments straight into your own Stripe account and keep all of your attendee data.

- One-off and recurring events with a drag and drop event page designer
- Paid, free, donation and tiered tickets, add-ons, promo codes, waitlists and shared capacity
- Stripe payouts, taxes and fees, invoices, offline payments and refunds
- Check-in with a phone camera or USB scanner, branded PDF tickets and attendee messaging
- Team roles, SSO (Google, Microsoft or any OIDC provider), 2FA, webhooks and a REST API

<div class="glance">
  <div><span>Port</span><b><code>8123</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>3 services</b></div>
  <div><span>Depends on</span><b>Postgres and Redis</b></div>
  <div><span>Difficulty</span><b class="difficulty intermediate">Intermediate</b></div>
  <div class="glance-links">
    <a href="https://github.com/HiEventsDev/Hi.Events"><i class="mdi mdi-github"></i>Project</a>
    <a href="https://hi.events/docs"><i class="mdi mdi-book-open-variant"></i>Docs</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Hi.Events
```yaml
services:
  hi-events:
    image: daveearley/hi.events-all-in-one:v2.0.0
    container_name: hi-events
    restart: unless-stopped
    ports:
      - 8123:80
    environment:
      - APP_KEY=CHANGE_ME # echo "base64:$(openssl rand -base64 32)"
      - JWT_SECRET=CHANGE_ME # openssl rand -base64 32
      - VITE_FRONTEND_URL=http://YOUR-SERVER-IP:8123
      - VITE_API_URL_CLIENT=http://YOUR-SERVER-IP:8123/api
      - VITE_API_URL_SERVER=http://localhost:80/api
      - VITE_STRIPE_PUBLISHABLE_KEY=
      - VITE_APP_NAME=Hi.Events
      - APP_FRONTEND_URL=http://YOUR-SERVER-IP:8123
      - APP_CDN_URL=http://YOUR-SERVER-IP:8123/storage
      - APP_DISABLE_REGISTRATION=false
      - APP_SAAS_MODE_ENABLED=false
      - LOG_CHANNEL=stderr
      - QUEUE_CONNECTION=redis
      - WEBHOOK_QUEUE_NAME=webhook-queue
      - FILESYSTEM_PUBLIC_DISK=public
      - FILESYSTEM_PRIVATE_DISK=local
      - DB_CONNECTION=pgsql
      - DATABASE_URL=postgresql://hi-events:CHANGE_ME@hi-events-db:5432/hi-events
      - REDIS_HOST=hi-events-redis
      - REDIS_PASSWORD=
      - REDIS_PORT=6379
      - MAIL_MAILER=log
      - MAIL_FROM_ADDRESS=tickets@example.com
      - MAIL_FROM_NAME=Hi.Events
      - STRIPE_PUBLIC_KEY=
      - STRIPE_SECRET_KEY=
      - STRIPE_WEBHOOK_SECRET=
    volumes:
      - /mnt/tank/configs/hi-events/storage:/app/backend/storage/app
    depends_on:
      hi-events-db:
        condition: service_healthy
      hi-events-redis:
        condition: service_healthy

  hi-events-db:
    image: postgres:17-alpine
    container_name: hi-events-db
    restart: unless-stopped
    environment:
      - POSTGRES_DB=hi-events
      - POSTGRES_USER=hi-events
      - POSTGRES_PASSWORD=CHANGE_ME
    volumes:
      - /mnt/tank/configs/hi-events/postgres:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U hi-events -d hi-events"]
      interval: 10s
      timeout: 5s
      retries: 5

  hi-events-redis:
    image: redis:7-alpine
    container_name: hi-events-redis
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/hi-events/redis:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
```

1. Generate the two secrets in the TrueNAS shell. `APP_KEY` needs the `base64:` prefix and `JWT_SECRET` does not:
    ```bash
    echo "base64:$(openssl rand -base64 32)"
    openssl rand -base64 32
    ```
1. Change `POSTGRES_PASSWORD` and the matching password inside `DATABASE_URL`. Avoid `@`, `:` and `/` in it, since it sits inside a URL
1. Replace every `YOUR-SERVER-IP:8123` with the address you will use to reach Hi.Events. Behind a reverse proxy, use your public URL, for example `https://tickets.example.com`
1. Leave `VITE_API_URL_SERVER` as `http://localhost:80/api`. It is used inside the container
1. Deploy the stack

> Never change `APP_KEY` once people have signed up. It encrypts 2FA and SSO secrets, and changing it breaks every pending invite and confirmation link.
{.is-danger}

> Port `8123` is also Home Assistant's default. If you run both, change the left side of the mapping and update all of the URLs to match.
{.is-warning}

# 2 · Configuration

## 2.1 Create Your Account
1. Open `http://YOUR-SERVER-IP:8123/auth/register` and create your account and organizer
1. Once your account exists, set `APP_DISABLE_REGISTRATION=true` and redeploy so nobody else can sign up

> If you invite teammates later, flip registration back on only while they accept.
{.is-info}

## 2.2 Email
The compose above uses the `log` mailer, so emails are written to the logs instead of being sent. Ticket emails, invites and password resets need real mail, so swap in your SMTP provider:

```yaml
      - MAIL_MAILER=smtp
      - MAIL_HOST=smtp.example.com
      - MAIL_PORT=587
      - MAIL_USERNAME=your-username
      - MAIL_PASSWORD=your-password
      - MAIL_FROM_ADDRESS=tickets@example.com
      - MAIL_FROM_NAME=Hi.Events
```

Port `465` uses implicit TLS and any other port uses STARTTLS automatically.

## 2.3 Stripe Payments
Free events work without Stripe. To sell paid tickets:
1. Copy your publishable key into both `STRIPE_PUBLIC_KEY` and `VITE_STRIPE_PUBLISHABLE_KEY`, and your secret key into `STRIPE_SECRET_KEY`
1. In the Stripe dashboard, add a webhook pointing at `https://YOUR-DOMAIN/api/public/webhooks/stripe`
1. Subscribe it to these events:
    - `payment_intent.succeeded`, `payment_intent.payment_failed`
    - `charge.succeeded`, `charge.updated`, `charge.refunded`
    - `refund.created`, `refund.updated`
    - `account.updated`, `payout.paid`, `payout.updated`
1. Copy the signing secret into `STRIPE_WEBHOOK_SECRET` and redeploy

> Stripe has to reach the webhook URL, so selling tickets means exposing Hi.Events through a reverse proxy or [Cloudflare Tunnel](/CloudflareTunnels) with HTTPS.
{.is-warning}

> Hi.Events is AGPL licensed with one extra term: the "Powered by Hi.Events" footer must stay unless you buy a commercial license. Reserved seating and the box office are Enterprise features that need a license key.
{.is-info}

# <img src="/youtube.png" class="tab-icon"> 3 · Video

*No video yet, check back soon!*
