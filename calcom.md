---
title: Cal.com
description: A guide to deploying Cal.com
published: true
date: 2026-04-15T17:41:45.728Z
tags: 
editor: markdown
dateCreated: 2026-03-07T11:20:08.256Z
---

# <img src="/cal-com.png" class="tab-icon"> What is Cal.diy (formerly Cal.com)?

**Cal.diy** is the open-source community edition of Cal.com, a scheduling platform that lets you create bookable event types, sync with your existing calendars, and accept payments through Stripe. In April 2026, Cal.com moved its commercial product to closed source and released Cal.diy under the MIT license as the self-hostable fork for the community. It's a self-hosted alternative to Calendly with full control over your data, making it ideal for consultants, creators, and freelancers who want a professional booking page without handing their schedule to a third party.

> **Migrating from the old `calcom/cal.com` image?** Back up your database directory first, then swap the image to `calcom/cal.diy:latest`. Your environment variables and data are compatible. Enterprise-only features (Teams, Organizations, Workflows, SSO/SAML, Insights) are no longer available in Cal.diy. See [Cal.com's announcement](https://cal.com/blog/calcom-v6-4) for full details.
{.is-warning}

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Cal.diy


```yaml
services:
  calcom-db:
    image: postgres:16
    container_name: calcom-db
    restart: unless-stopped
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    volumes:
      - /mnt/tank/configs/calcom/db:/var/lib/postgresql/data
    healthcheck:
      test:
        - CMD-SHELL
        - pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}
      interval: 5s
      timeout: 5s
      retries: 10

  calcom:
    image: calcom/cal.diy:latest
    container_name: calcom
    restart: unless-stopped
    depends_on:
      - calcom-db
    ports:
      - "${CALCOM_PORT:-3000}:3000"
    environment:
      # --- Core Settings ---
      - NEXT_PUBLIC_WEBAPP_URL=${CALCOM_URL}
      - NEXTAUTH_URL=${CALCOM_URL}
      - NEXTAUTH_SECRET=${NEXTAUTH_SECRET}
      - CALENDSO_ENCRYPTION_KEY=${CALENDSO_ENCRYPTION_KEY}
      - CALCOM_TELEMETRY_DISABLED=1
      # --- Database ---
      - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@calcom-db:5432/${POSTGRES_DB}
      - DATABASE_DIRECT_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@calcom-db:5432/${POSTGRES_DB}
      # --- Email (SMTP) ---
      - EMAIL_FROM=${EMAIL_FROM}
      - EMAIL_SERVER_HOST=${EMAIL_SERVER_HOST}
      - EMAIL_SERVER_PORT=${EMAIL_SERVER_PORT:-587}
      - EMAIL_SERVER_USER=${EMAIL_SERVER_USER}
      - EMAIL_SERVER_PASSWORD=${EMAIL_SERVER_PASSWORD}
      # --- Stripe (for paid bookings) ---
      - NEXT_PUBLIC_STRIPE_PUBLIC_KEY=${STRIPE_PUBLIC_KEY}
      - STRIPE_PRIVATE_KEY=${STRIPE_PRIVATE_KEY}
      - STRIPE_WEBHOOK_SECRET=${STRIPE_WEBHOOK_SECRET}
      - STRIPE_CLIENT_ID=${STRIPE_CLIENT_ID}
      - PAYMENT_FEE_FIXED=${PAYMENT_FEE_FIXED:-0}
      - PAYMENT_FEE_PERCENTAGE=${PAYMENT_FEE_PERCENTAGE:-0}
      # --- Google Calendar OAuth ---
      - GOOGLE_API_CREDENTIALS=${GOOGLE_API_CREDENTIALS}
      # --- Push Notifications (VAPID) ---
      - NEXT_PUBLIC_VAPID_PUBLIC_KEY=${VAPID_PUBLIC_KEY}
      - VAPID_PRIVATE_KEY=${VAPID_PRIVATE_KEY}
    volumes:
      - /mnt/tank/configs/calcom/config:/app/config
```

## Environment Variables (.env)

Create a `.env` file in the same directory as your compose file (Dockge handles this automatically in the environment variables section):

```bash
# --- Database ---
POSTGRES_USER=calcom
POSTGRES_PASSWORD=CHANGE_ME_STRONG_PASSWORD
POSTGRES_DB=calcom

# --- Cal.diy Core ---
CALCOM_URL=https://cal.example.com
CALCOM_PORT=3000
NEXTAUTH_SECRET=CHANGE_ME_GENERATE_WITH_OPENSSL
CALENDSO_ENCRYPTION_KEY=CHANGE_ME_GENERATE_WITH_OPENSSL

# --- Email (SMTP) ---
EMAIL_FROM=cal@example.com
EMAIL_SERVER_HOST=smtp.example.com
EMAIL_SERVER_PORT=587
EMAIL_SERVER_USER=your_smtp_user
EMAIL_SERVER_PASSWORD=your_smtp_password

# --- Stripe (for paid bookings) ---
STRIPE_PUBLIC_KEY=pk_live_XXXXX
STRIPE_PRIVATE_KEY=sk_live_XXXXX
STRIPE_WEBHOOK_SECRET=whsec_XXXXX
STRIPE_CLIENT_ID=ca_XXXXX
PAYMENT_FEE_FIXED=0
PAYMENT_FEE_PERCENTAGE=0

# --- Google Calendar OAuth ---
GOOGLE_API_CREDENTIALS={"client_id":"XXXXX.apps.googleusercontent.com","client_secret":"XXXXX","redirect_uris":["https://cal.example.com/api/integrations/googlecalendar/callback"]}

# --- Push Notifications (VAPID) ---
VAPID_PUBLIC_KEY=CHANGE_ME_GENERATE_WITH_WEB_PUSH
VAPID_PRIVATE_KEY=CHANGE_ME_GENERATE_WITH_WEB_PUSH
```

1. Replace all `CHANGE_ME` values with your own credentials
2. Replace `cal.example.com` with your actual domain (e.g., `cal.serversatho.me`)
3. Generate secrets: run `openssl rand -base64 32` for `NEXTAUTH_SECRET` and `openssl rand -base64 24` for `CALENDSO_ENCRYPTION_KEY`
4. Generate VAPID keys: run `npx web-push generate-vapid-keys` and copy the public/private key pair

> In Dockge, paste the compose YAML on the right and fill in the environment variables in the **Environment Variables** section at the bottom. Dockge creates the `.env` file for you.
{.is-success}

> If your pool is named something besides `tank`, change the left side of the volume paths accordingly.
{.is-info}

> Cal.diy uses `latest` here for simplicity, but consider pinning to a specific version tag (e.g., `calcom/cal.diy:v6.2.0`) for more predictable updates. Check available tags at [DockerHub](https://hub.docker.com/r/calcom/cal.diy/tags).
{.is-info}

# 2 · Configuration

## 2.1 Generating Secrets

Before deploying, generate your required secret keys from a terminal:

```bash
# Generate NEXTAUTH_SECRET
openssl rand -base64 32

# Generate CALENDSO_ENCRYPTION_KEY
openssl rand -base64 24

# Generate VAPID keys for push notifications
npx web-push generate-vapid-keys
```

Copy each output and paste it into the corresponding environment variable in your compose file.

## 2.2 Reverse Proxy Setup

Cal.diy should be accessed over HTTPS. Set up a reverse proxy in **Nginx Proxy Manager** or your preferred reverse proxy:

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

To sync Cal.diy with your Google Calendar:

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

Once deployed, go to **Settings → Integrations** in Cal.diy and click **Connect** next to Google Calendar to complete the OAuth flow.

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
7. In Cal.diy, go to **Apps → App Store** → search for **Stripe** → Install
8. Create or edit an event type → under **Apps**, enable Stripe and set your price

> Stripe also supports Apple Pay and Google Pay. Enable these in your Stripe Dashboard under **Settings → Payments → Payment Methods** for additional checkout options.
{.is-success}

## 2.5 Creating Consultation Event Types

After initial setup, create your bookable event types:

1. Go to **Event Types** in Cal.diy
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
| `NEXT_PUBLIC_WEBAPP_URL` | Public URL of your Cal.diy instance |
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
| `NEXT_PUBLIC_VAPID_PUBLIC_KEY` | VAPID public key for push notifications |
| `VAPID_PRIVATE_KEY` | VAPID private key for push notifications |
{.dense}

GitHub repository: [github.com/calcom/cal.diy](https://github.com/calcom/cal.diy)

# <img src="/youtube.png" class="tab-icon"> 3 · Video