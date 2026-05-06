---
title: Dockflare
description: A guide to deploying Dockflare
published: true
date: 2026-05-06T11:57:05.058Z
tags: 
editor: markdown
dateCreated: 2026-05-01T19:51:02.015Z
---

# <img src="/dockflare.png" class="tab-icon"> What is DockFlare?

**DockFlare** is a self-hosted ingress and access-control plane for Cloudflare Tunnels. It watches your Docker containers and translates simple labels into Cloudflare Tunnel routes, DNS records, and Zero Trust Access policies — so a new public hostname for any container is one label away, with no clicking through the Cloudflare dashboard.


Starting with v3.1.0, DockFlare also ships a self-hosted **Sovereign Email Suite** that uses Cloudflare Email Routing, Workers, R2, and KV as a delivery layer while keeping mailbox data, attachments, and full-text search local on your server.



# 0 · About the Email Suite (Read Me First)

The Email Suite is not a traditional self-hosted mail server. It is a **webmail front-end** to Cloudflare Email Routing + Workers + R2. 

## What it IS

- A way to receive mail at `you@yourdomain.com` without running an MTA
- A local SQLite-backed mail archive with full-text search
- An installable PWA (Progressive Web App) for reading and composing mail on desktop and mobile
- A way to send mail with proper DKIM/SPF/DMARC alignment, signed by Cloudflare

## What it is NOT

- **There is no IMAP server.** Thunderbird, Apple Mail, Outlook, K-9 Mail, native iOS/Android mail apps — none of these can connect to your DockFlare mailbox. The webmail PWA is the only client.
- **There is no SMTP submission server.** You can't point an external client at your domain and have it send. Sending only works through the webmail.
- **There is no MTA you can run other tools against.** No Sieve filtering. No fetchmail/getmail integration. No external archiver.

## The Free-Tier Outbound Restriction

Cloudflare's `send_email` Worker binding (the thing DockFlare uses to send mail) is gated by default. On the free tier, you can only send mail to **destination addresses you have verified** through Cloudflare's dashboard.

In practice this means: when you try to email someone for the first time, Cloudflare sends them a verification email asking them to click a link to authorize you to send to them. 

**Workers Paid plan ($5/month)** lifts this restriction: after provisioning four `cf-bounce.<yourdomain>` DNS records and waiting for them to lock, your domain is treated as an authenticated sender and you can send to anyone. 


# 1 · Architecture and Prerequisites

## 1.1 The DockFlare Tunnel Model

When you complete the setup wizard, three things happen automatically:

1. DockFlare calls Cloudflare's API and creates a brand-new tunnel
2. Cloudflare returns a tunnel token
3. DockFlare spawns a `cloudflared` container on your Docker host using that token

From then on, every container you label with `dockflare.enable=true` gets a route added to **DockFlare's tunnel**. You can run DockFlare alongside existing Cloudflare Tunnels without conflict — each tunnel is independent.

## 1.2 The Shared Network

DockFlare spawns the `cloudflared` container at runtime via the Docker API, so the network it attaches that container to has to **already exist by a known name**. That's why the compose declares `cloudflare-net` as an external network.

Create the network from a TrueNAS shell before deploying:

```bash
docker network create cloudflare-net
```

## 1.3 Cloudflare Account Items

Gather these from your Cloudflare account before deploying:

| Item | Where to find it |
|------|------------------|
| Account ID | Any zone overview page, right sidebar |
| Zone ID | Your domain's overview page, right sidebar |
| API token | My Profile → API Tokens → Create Token |


Your API token needs the base DockFlare scopes plus the Email Suite scopes layered on top. The names below match the exact labels in Cloudflare's token creation UI.

**Base DockFlare scopes (for tunnel and Access management):**

- **Account** → Cloudflare Tunnel: Edit
- **Account** → Access: Apps and Policies: Edit
- **Account** → Access: Organizations, Identity Providers, and Groups: Edit
- **Account** → Access: Service Tokens: Edit
- **Account** → Account Settings: Read
- **Zone** → Zone: Read
- **Zone** → DNS: Edit

**Additional scopes for the Email Suite:**

- **Account** → Email Routing Addresses: Edit *(managing destination forwarding addresses)*
- **Account** → Workers Scripts: Edit *(deploying inbound/outbound workers)*
- **Account** → Workers KV Storage: Edit *(quota enforcement at the edge)*
- **Account** → Workers R2 Storage: Edit *(transit buckets for inbound mail)*
- **Zone** → Email Routing Rules: Edit *(activating routing and managing rules)*
- **User** → User Details: Read *(allows DockFlare to call Cloudflare's `/tokens/verify` endpoint — without this, email domain setup fails with `401 Unauthorized`)*

For **Account Resources**, set this to **Include → All accounts** during initial setup. You can tighten to a specific account later if everything works, but loose scoping rules out a class of `/tokens/verify` 401 errors that have been observed with account-scoped tokens.

For **Zone Resources**, **Include → Specific zone → [your domain]** is fine to keep tight.

> 
> Restrict the token's scope to just what DockFlare needs. Do not grant a token with these scopes access to all your accounts or zones unless you actually need that.
{.is-warning}

> 
> **Your domain must use Cloudflare DNS.** If your domain is registered elsewhere, point its nameservers to Cloudflare before continuing — Email Routing only works on zones served by Cloudflare's nameservers.
{.is-warning}

> 
> **Email Routing takes ownership of your zone's MX records.** If you currently use Google Workspace, Fastmail, or any other mail provider for this domain, that mail flow has to be retired before enabling DockFlare email — you cannot run both at the same time.
{.is-danger}

## 1.4 Activate R2 Object Storage

DockFlare uses Cloudflare R2 as the inbound mail buffer. R2 is opt-in on every Cloudflare account, so you have to enable it before the email setup will work.

If you skip this step, the **Email Management** page in DockFlare will show R2 Storage with a red ❌ in the Permissions Required panel and the **Setup Email for Domain** button will stay disabled.

1. Open the Cloudflare dashboard
2. In the left sidebar, click **R2 Object Storage**
3. Click **Enable R2** and follow the activation flow
4. Add a payment method if prompted

> 
> **R2 has a 10 GB free tier**, but Cloudflare requires a payment method on file before activation. You will not be charged unless you exceed the free tier — for personal email this is essentially impossible. The mail manager pulls messages out of R2 into local storage almost immediately, so R2 is just a transit buffer.
{.is-info}

5. Wait about 30 seconds for the activation to propagate
6. Return to the DockFlare **Email** page and click **Check Permissions**

# <img src="/docker.png" class="tab-icon"> 2 · Deploy DockFlare



```yaml
services:
  docker-socket-proxy:
    image: tecnativa/docker-socket-proxy:v0.4.1
    container_name: docker-socket-proxy
    restart: unless-stopped
    logging:
      driver: "none"
    environment:
      - DOCKER_HOST=unix:///var/run/docker.sock
      - CONTAINERS=1
      - EVENTS=1
      - NETWORKS=1
      - IMAGES=1
      - POST=1
      - PING=1
      - INFO=1
      - EXEC=1
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

  dockflare-init:
    image: alpine:3.20
    command: ["sh", "-c", "chown -R 65532:65532 /app/data /data"]
    volumes:
      - /mnt/tank/configs/dockflare/data:/app/data
      - /mnt/tank/configs/dockflare/mail:/data
    restart: "no"

  dockflare:
    image: alplat/dockflare:stable
    container_name: dockflare
    restart: unless-stopped
    ports:
      - "5000:5000"
    labels:
      - dockflare.enable=true
      - dockflare.hostname=dockflare.${DOMAIN}
      - dockflare.service=http://dockflare:5000
    volumes:
      - /mnt/tank/configs/dockflare/data:/app/data
    environment:
      - REDIS_URL=redis://redis:6379/0
      - REDIS_DB_INDEX=0
      - DOCKER_HOST=tcp://docker-socket-proxy:2375
    depends_on:
      docker-socket-proxy:
        condition: service_started
      dockflare-init:
        condition: service_completed_successfully
      redis:
        condition: service_started
    networks:
      - cloudflare-net
      - default

  redis:
    image: redis:7-alpine
    container_name: dockflare-redis
    restart: unless-stopped
    command: ["redis-server", "--save", "", "--appendonly", "no"]
    logging:
      driver: "none"
    volumes:
      - /mnt/tank/configs/dockflare/redis:/data

  dockflare-mail-manager:
    image: alplat/dockflare-mail-manager:stable
    container_name: dockflare-mail-manager
    restart: unless-stopped
    environment:
      - DOCKFLARE_MASTER_URL=http://dockflare:5000
      - MAIL_DATA_PATH=/data
    volumes:
      - /mnt/tank/configs/dockflare/mail:/data
    depends_on:
      dockflare:
        condition: service_started
    networks:
      - cloudflare-net
      - default

  dockflare-webmail:
    image: alplat/dockflare-webmail:stable
    container_name: dockflare-webmail
    restart: unless-stopped
    environment:
      - DOCKFLARE_MASTER_URL=https://dockflare.${DOMAIN}
    labels:
      - dockflare.enable=true
      - dockflare.hostname=mail.${DOMAIN}
      - dockflare.service=http://dockflare-webmail:80
    depends_on:
      dockflare-mail-manager:
        condition: service_started
    networks:
      - cloudflare-net
      - default

networks:
  cloudflare-net:
    name: cloudflare-net
    external: true
```

> 
> The `dockflare-init` container is a one-shot that fixes ownership on the bind-mounted data directories before the main containers start. It exits after running, which is normal — Dockge will show it as "exited." After the first successful deploy, you can remove the `dockflare-init` block from `depends_on` (and optionally delete the service entirely) to keep your Dockge stack list clean.
{.is-info}

> 
> Both the DockFlare admin UI (`dockflare.yourdomain.com`) and the webmail (`mail.yourdomain.com`) get exposed via Cloudflare Tunnel automatically because of the `dockflare.enable=true` labels. **Add a Cloudflare Access policy to the admin hostname before going live** — without one it would be publicly reachable on the internet.
{.is-danger}

## 2.1 env Variables
Set `DOMAIN=yourdomain.com` in the stack's environment variables.

# 3 · Initial Setup Wizard

1. Open `http://<truenas-ip>:5000`
2. Walk through the four-step wizard:
   - **Web Access:** create the admin user
   - **Cloudflare:** enter your Account ID, Zone ID, and API token
   - **Tunnel:** name the tunnel DockFlare will create — `dockflare-tunnel` is fine
   - **Finalize:** confirm and let DockFlare provision the tunnel
3. Confirm the dashboard loads and the tunnel shows healthy

After the tunnel is up, both `dockflare.yourdomain.com` and `mail.yourdomain.com` will start resolving to your DockFlare instance.

# 4 · Configure Your Email Domain

1. In the DockFlare UI, navigate to **Email** in the top navigation
2. Click **Add Domain** and enter your domain (e.g. `yourdomain.com`)
3. Click **One-Click Setup** — DockFlare will:
   - Enable Cloudflare Email Routing on the zone
   - Create the R2 bucket for inbound message buffering
   - Create the KV namespace for alias and quota lookups
   - Deploy the inbound and outbound Workers
   - Write the MX, SPF, DMARC, and DKIM records into your DNS
4. Click **Add Mailbox** and create your mailbox (e.g. `you`)
5. Set a per-mailbox storage quota
6. Click **Save**

> 
> If the one-click setup fails partway through, use **Repair DNS** on the domain detail page to re-apply any missing records. Worker deployments can also be redeployed from the same page.
{.is-info}

## 4.1 Verify Destination Addresses (Free Tier Only)

Before you can send mail to anyone on the free tier, you have to register their address as a verified destination in Cloudflare.

1. Cloudflare dashboard → your domain → **Email** → **Email Routing** → **Destination Addresses**
2. Click **Add Destination Address** and enter the recipient's email
3. Cloudflare emails the recipient a verification link
4. The recipient must click the link to authorize you sending to them

This is a hard requirement of Cloudflare's `send_email` Worker binding, not a DockFlare limitation. Each recipient is a one-time per-account setup.



## 4.2 Lift the Restriction with Workers Paid

To send to any address without per-recipient verification, upgrade to the **Workers Paid plan** ($5/month):

1. Cloudflare dashboard → **Workers & Pages** → **Plans** → upgrade to Paid
2. Cloudflare provisions four `cf-bounce.yourdomain.com` records (MX, SPF, DKIM, DMARC) into your zone
3. Wait ~5 minutes for the records to show "Locked" status
4. Outbound mail to any destination should now work



# 5 · Access the Webmail

The webmail is automatically published through Cloudflare Tunnel at `https://mail.yourdomain.com` because of the labels in the compose. Open that URL from any browser, on any network.

> 
> Lock down `mail.yourdomain.com` with a Cloudflare Access policy in the DockFlare UI before sending real mail. Without one, anyone who guesses the hostname could hit your login page.
{.is-warning}

## 5.1 Install as a PWA

In a Chromium-based browser or Safari, open `https://mail.yourdomain.com` and look for the install icon in the address bar (or **Add to Home Screen** on mobile). Click **Install** — the webmail now runs as a standalone app with desktop and mobile push notifications.



# 6 · Disposable Email Aliases

DockFlare v3.1.1 added a disposable alias system enforced at the Cloudflare Worker layer. From the webmail Settings panel:

1. Click **Aliases → Create Alias**
2. Choose a generation style: `word-word-num`, `word-num`, `uuid-short`, or define a custom local-part
3. Optionally set an expiry date and a label/description
4. Save

Replies to an alias-received email pre-select that alias as the sender. Expired aliases are deactivated hourly.

| Limit | Value |
|-------|-------|
| Alias creations per hour | 20 |
| Total aliases per mailbox | 100 |
| Outbound rate (per sender, free tier) | 50/hr, 200/day |




# <img src="/youtube.png" class="tab-icon"> 7 · Video

