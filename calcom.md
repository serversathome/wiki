---
title: Cal.com
description: A guide to deploying Cal.com
published: true
date: 2026-03-07T11:21:00.310Z
tags: 
editor: markdown
dateCreated: 2026-03-07T11:20:08.256Z
---

# <img src="/cal-com.png" class="tab-icon"> What is Cal.com?

**Cal.com** is an open-source scheduling platform that lets you create bookable event types, sync with your existing calendars, and accept payments through Stripe. It's a self-hosted alternative to Calendly with full control over your data, making it ideal for consultants, creators, and freelancers who want a professional booking page without handing their schedule to a third party.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Cal.com

> Cal.com requires several environment variables to function properly. Make sure you generate unique secrets for `NEXTAUTH_SECRET` and `CALENDSO_ENCRYPTION_KEY` before deploying. See section 2.1 for instructions.
{.is-warning}

```yaml
services:
  calcom-db:
    image: postgres:16
    container_name: calcom-db
    restart: unless-stopped
    environment:
      - POSTGRES_USER=calcom
      - POSTGRES_PASSWORD=CHANGE_ME_DB_PASSWORD
      - POSTGRES_DB=calcom
    volumes:
      - /mnt/tank/configs/calcom/db:/var/lib/postgresql/data

  calcom:
    image: calcom.docker.scarf.sh/calcom/cal.com:latest
    container_name: calcom
    restart: unless-stopped
    depends_on:
      - calcom-db
    ports:
      - "3000:3000"
    environment:
      # --- Core Settings ---
      - NEXT_PUBLIC_WEBAPP_URL=https://cal.example.com
      - NEXTAUTH_URL=https://cal.example.com
      - NEXTAUTH_SECRET=CHANGE_ME_GENERATE_WITH_OPENSSL
      - CALENDSO_ENCRYPTION_KEY=CHANGE_ME_GENERATE_WITH_OPENSSL
      - CALCOM_TELEMETRY_DISABLED=1
      # --- Database ---
      - DATABASE_URL=postgresql://calcom:CHANGE_ME_DB_PASSWORD@calcom-db:5432/calcom
      - DATABASE_DIRECT_URL=postgresql://calcom:CHANGE_ME_DB_PASSWORD@calcom-db:5432/calcom
      # --- Email (SMTP) ---
      - EMAIL_FROM=cal@example.com
      - EMAIL_SERVER_HOST=smtp.example.com
      - EMAIL_SERVER_PORT=587
      - EMAIL_SERVER_USER=your_smtp_user
      - EMAIL_SERVER_PASSWORD=your_smtp_password
      # --- Stripe (for paid bookings) ---
      - NEXT_PUBLIC_STRIPE_PUBLIC_KEY=pk_live_XXXXX
      - STRIPE_PRIVATE_KEY=sk_live_XXXXX
      - STRIPE_WEBHOOK_SECRET=whsec_XXXXX
      - STRIPE_CLIENT_ID=ca_XXXXX
      - PAYMENT_FEE_FIXED=0
      - PAYMENT_FEE_PERCENTAGE=0
      # --- Google Calendar OAuth ---
      - GOOGLE_API_CREDENTIALS={"client_id":"XXXXX.apps.googleusercontent.com","client_secret":"XXXXX","redirect_uris":["https://cal.example.com/api/integrations/googlecalendar/callback"]}
      # --- License (optional, for enterprise features) ---
      # - CALCOM_LICENSE_KEY=
    volumes:
      - /mnt/tank/configs/calcom/config:/app/config
```

1. Replace all `CHANGE_ME` values with your own credentials
2. Replace `cal.example.com` with your actual domain (e.g., `cal.example.com`)
3. Generate secrets using `openssl rand -base64 32` for `NEXTAUTH_SECRET` and `openssl rand -base64 24` for `CALENDSO_ENCRYPTION_KEY`

> If your pool is named something besides `tank`, change the left side of the volume paths accordingly.
{.is-info}

# 2 · Configuration

## 2.1 Generating Secrets

Before deploying, generate your two required secret keys from a terminal:

```bash
# Generate NEXTAUTH_SECRET
openssl rand -base64 32

# Generate CALENDSO_ENCRYPTION_KEY
openssl rand -base64 24
```

Copy each output and paste it into the corresponding environment variable in your compose file.

## 2.2 Reverse Proxy Setup

Cal.com should be accessed over HTTPS. Set up a reverse proxy in **Nginx Proxy Manager** or your preferred reverse proxy:

| Setting | Value |
|---------|-------|
| **Scheme** | http |
| **Forward Hostname** | calcom (or container IP) |
| **Forward Port** | 3000 |
| **SSL** | Request a new SSL certificate or use Cloudflare |

If you're using a **Cloudflare Tunnel**, point the tunnel to `http://calcom:3000` and set your public hostname to your desired subdomain.

> Make sure `NEXT_PUBLIC_WEBAPP_URL` and `NEXTAUTH_URL` both match your final public URL exactly (including `https://`). Mismatches here will cause login and callback failures.
{.is-danger}

## 2.3 Google Calendar Integration

To sync Cal.com with your Google Calendar:

1. Go to the [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select an existing one
3. Enable the **Google Calendar API**
4. Go to **OAuth consent screen** → configure as External
5. Add scopes: `auth/calendar.events` and `auth/calendar.readonly`
6. Go to **Credentials** → Create **OAuth 2.0 Client ID**
7. Set Application type to **Web application**
8. Add Authorized redirect URIs:
   - `https://cal.example.com/api/integrations/googlecalendar/callback`
   - `https://cal.example.com/api/auth/callback/google`
9. Copy the Client ID and Client Secret into the `GOOGLE_API_CREDENTIALS` environment variable in your compose file

> The `GOOGLE_API_CREDENTIALS` value must be valid JSON on a single line. Double-check your quotes and escaping.
{.is-warning}

Once deployed, go to **Settings → Integrations** in Cal.com and click **Connect** next to Google Calendar to complete the OAuth flow.

## 2.4 Stripe Integration (Paid Bookings)

To accept payments when someone books your time:

1. Create a [Stripe account](https://stripe.com) if you don't have one
2. In the Stripe Dashboard, go to **Developers → API Keys** and copy your Publishable Key (`pk_live_...`) and Secret Key (`sk_live_...`)
3. Go to **Developers → Webhooks** → Add an endpoint
   - URL: `https://cal.example.com/api/integrations/stripe/webhook`
   - Events: Select all `payment_intent` and `setup_intent` events
   - Copy the Webhook Signing Secret (`whsec_...`)
4. For `STRIPE_CLIENT_ID`, go to **Settings → Connect** in Stripe and copy your Platform Client ID (`ca_...`)
5. Add all four values to your compose environment variables
6. Deploy/redeploy your stack
7. In Cal.com, go to **Apps → App Store** → search for **Stripe** → Install
8. Create or edit an event type → under **Apps**, enable Stripe and set your price

> Stripe also supports Apple Pay and Google Pay. Enable these in your Stripe Dashboard under **Settings → Payments → Payment Methods** for additional checkout options.
{.is-success}

## 2.5 Creating Consultation Event Types

After initial setup, create your bookable event types:

1. Go to **Event Types** in Cal.com
2. Click **New Event Type**
3. Configure:
   - **Title**: e.g., "Homelab Consultation"
   - **Duration**: 30 min, 60 min, etc.
   - **Price**: Set via Stripe integration
   - **Location**: Cal Video (built-in), Zoom, Google Meet, etc.
4. Set your **Availability** schedule (days/hours you're available)
5. Add **Buffer time** between meetings to prevent back-to-back bookings
6. Set **Daily/Weekly booking limits** to control your schedule
7. Click **Save**

Your booking page will be available at `https://cal.example.com/your-username`.

## 2.6 Environment Variables Reference

| Variable | Description |
|----------|-------------|
| `NEXT_PUBLIC_WEBAPP_URL` | Public URL of your Cal.com instance |
| `NEXTAUTH_URL` | Should match `NEXT_PUBLIC_WEBAPP_URL` |
| `NEXTAUTH_SECRET` | Random secret for session encryption |
| `CALENDSO_ENCRYPTION_KEY` | Random key for data encryption |
| `DATABASE_URL` | PostgreSQL connection string |
| `CALCOM_TELEMETRY_DISABLED` | Set to `1` to disable telemetry |
| `EMAIL_FROM` | Sender address for booking confirmations |
| `EMAIL_SERVER_HOST` | SMTP server hostname |
| `NEXT_PUBLIC_STRIPE_PUBLIC_KEY` | Stripe publishable key (`pk_live_...`) |
| `STRIPE_PRIVATE_KEY` | Stripe secret key (`sk_live_...`) |
| `STRIPE_WEBHOOK_SECRET` | Stripe webhook signing secret (`whsec_...`) |
| `STRIPE_CLIENT_ID` | Stripe Connect client ID (`ca_...`) |
| `GOOGLE_API_CREDENTIALS` | JSON string with Google OAuth credentials |
{.dense}

Full documentation: [cal.com/docs/self-hosting](https://cal.com/docs/self-hosting)

# <img src="/youtube.png" class="tab-icon"> 3 · Video
