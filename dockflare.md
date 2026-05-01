---
title: Dockflare
description: A guide to deploying Dockflare
published: false
date: 2026-05-01T19:51:02.015Z
tags: 
editor: markdown
dateCreated: 2026-05-01T19:51:02.015Z
---

# <img src="/dockflare.png" class="tab-icon"> What is DockFlare?

**DockFlare** is a self-hosted ingress and access-control plane for Cloudflare Tunnels. It watches your Docker containers and translates simple labels into Cloudflare Tunnel routes, DNS records, and Zero Trust Access policies — so a new public hostname for any container is one label away, with no clicking through the Cloudflare dashboard.

Starting with v3.1.0, DockFlare also ships a fully self-hosted **Sovereign Email Suite** that uses Cloudflare Email Routing, Workers, R2, and KV as a stateless delivery layer while keeping your mailbox data, attachments, and full-text search index local on your own server. You get a real `you@yourdomain.com` mailbox without ever running an SMTP server.

> 
> DockFlare creates and manages **its own dedicated Cloudflare Tunnel**. During setup, you give it a name (like `dockflare-tunnel`) and DockFlare provisions the tunnel via the Cloudflare API, then spawns a `cloudflared` container on the host to run it. If you already have other tunnels for other services, they keep working independently — DockFlare doesn't touch them.
{.is-info}

# 1 · Architecture and Prerequisites

## 1.1 The DockFlare Tunnel Model

When you complete the setup wizard, three things happen automatically:

1. DockFlare calls Cloudflare's API and creates a brand-new tunnel
2. Cloudflare returns a tunnel token
3. DockFlare spawns a `cloudflared` container on your Docker host using that token

From then on, every container you label with `dockflare.enable=true` gets a route added to **DockFlare's tunnel**. You manage everything through the DockFlare UI or container labels — never through the Cloudflare dashboard.

This means you can run DockFlare alongside existing tunnels without conflict. Each tunnel is independent.

## 1.2 The Shared Network

DockFlare creates the `cloudflared` container outside the compose lifecycle (it spawns it via the Docker API at runtime), so the network it attaches that container to has to **already exist by a known name**. That's why the compose declares `cloudflare-net` as an external network — DockFlare's runtime-created cloudflared container joins it to reach the DockFlare-managed services.

Create the network from a TrueNAS shell before deploying:

```bash
docker network create cloudflare-net
```

That's the only manual network step. Everything else is handled by the compose.

## 1.3 Cloudflare Account Items

Gather these from your Cloudflare account before deploying:

| Item | Where to find it |
|------|------------------|
| Account ID | Any zone overview page, right sidebar |
| Zone ID | Your domain's overview page, right sidebar |
| API token | My Profile → API Tokens → Create Token |
{.dense}

Your API token needs the base DockFlare scopes plus the Email Suite scopes layered on top. The names below match the exact labels shown in Cloudflare's token creation UI.

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

**Account Resources** should be set to **Include → Specific account → [your account]** and **Zone Resources** should be set to **Include → Specific zone → [your domain]**. The `/tokens/verify` endpoint is finicky about these scopings — using "All accounts" can also cause the 401 error.

> 
> Restrict the token's **Account Resources** and **Zone Resources** to only the specific account and domains you intend to use with DockFlare. Do not grant it access to your whole Cloudflare account.
{.is-warning}

> 
> **Your domain must use Cloudflare DNS.** If your domain is registered elsewhere, point its nameservers to Cloudflare before continuing — Email Routing only works on zones served by Cloudflare's nameservers.
{.is-warning}

> 
> **Email Routing takes ownership of your zone's MX records.** If you currently use Google Workspace, Fastmail, or any other mail provider for this domain, that mail flow has to be retired before enabling DockFlare email — you cannot run both at the same time.
{.is-danger}

> 
> **Outbound mail uses Cloudflare Email Sending**, which is currently in Beta and may require requesting access. If your one-click setup completes but outbound delivery fails, check whether your account has Email Sending enabled.
{.is-info}

## 1.4 Activate R2 Object Storage

DockFlare uses Cloudflare R2 as the inbound mail buffer — every message hits R2 first, then the mail manager pulls it into your local SQLite database. R2 is opt-in on every Cloudflare account, so you have to enable it before the email setup will work.

If you skip this step, the **Email Management** page in DockFlare will show R2 Storage with a red ❌ in the Permissions Required panel and the **Setup Email for Domain** button will stay disabled.

1. Open the Cloudflare dashboard
2. In the left sidebar, click **R2 Object Storage**
3. Click **Enable R2** and follow the activation flow
4. Add a payment method if prompted

> 
> **R2 has a 10 GB free tier**, but Cloudflare requires a payment method on file before activation. You will not be charged unless you exceed the free tier — for personal email this is essentially impossible. The mail manager pulls messages out of R2 into local storage almost immediately, so R2 is just a transit buffer, not where mail accumulates.
{.is-info}

5. Wait about 30 seconds for the activation to propagate
6. Return to the DockFlare **Email** page and click **Check Permissions**
7. R2 Storage should now show ✅ along with the other three checks

If R2 still shows red after activation, the issue is your API token rather than the account. Open **My Profile → API Tokens** in Cloudflare, edit the DockFlare token, and confirm it has **Account → Workers R2 Storage: Edit** as listed in section 1.3. Save the token — the existing token string keeps working, no need to regenerate it in DockFlare.

# <img src="/docker.png" class="tab-icon"> 2 · Deploy DockFlare

In Dockge, create a new stack at `/mnt/tank/stacks/dockflare/`. Paste this compose file and set `DOMAIN=yourdomain.com` in the stack's environment variables (Dockge UI → Environment, or a `.env` file in the stack folder). The `${DOMAIN}` variable feeds three places.

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
> The DockFlare master, mail manager, and webmail all run as user `65532:65532`. The `dockflare-init` one-shot fixes ownership on both bind-mounted directories before the main containers start.
{.is-info}

# 3 · Initial Setup Wizard

1. Open `http://<truenas-ip>:5000` (the master's port mapping is exposed on LAN for the wizard)
2. Walk through the four-step wizard:
   - **Step 1 (Web Access):** create the admin user
   - **Step 2 (Cloudflare):** enter your Account ID, Zone ID, and API token
   - **Step 3 (Tunnel):** name the tunnel DockFlare will create — `dockflare-tunnel` is fine, or pick something descriptive. Leave the optional Zone ID and Other Zones fields blank unless you have specific needs. The 28800-second grace period default is sensible.
   - **Step 4 (Finalize):** confirm and let DockFlare provision the tunnel
3. DockFlare will create the tunnel via the Cloudflare API and spawn a `cloudflared` container on the host
4. Confirm the dashboard loads and the tunnel shows healthy

After the tunnel is up, both `dockflare.yourdomain.com` and `mail.yourdomain.com` will start resolving to your DockFlare instance.

> 
> Both the DockFlare admin UI (`dockflare.yourdomain.com`) and the webmail (`mail.yourdomain.com`) get exposed via Cloudflare Tunnel automatically because of the `dockflare.enable=true` labels. **Add a Cloudflare Access policy to the admin hostname before going live** — without one it would be publicly reachable on the internet.
{.is-danger}

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

## 4.1 Test Inbound and Outbound Mail

From an external account (Gmail, Outlook, anything not on this domain):

1. Send a test message to `you@yourdomain.com`
2. The message should appear in the webmail within a few seconds
3. Reply to that message
4. Check the original headers on the receiving end — confirm `SPF=PASS`, `DKIM=PASS`, `DMARC=PASS`

> 
> The first outbound message to a major provider is the real deliverability test. Cloudflare's `send_email` Worker binding signs DKIM at the edge, but if any of the auth records are missing or wrong, Gmail will tell you fast.
{.is-success}

# 5 · Access the Webmail

The webmail is automatically published through Cloudflare Tunnel at `https://mail.yourdomain.com` because of the labels in the compose. There is no separate port to remember — open that URL from any browser, on any network, and log in with your mailbox credentials.

> 
> Lock down `mail.yourdomain.com` with a Cloudflare Access policy in the DockFlare UI before sending real mail. Without one, anyone who guesses the hostname could hit your login page.
{.is-warning}

## 5.1 Install as a PWA

1. In a Chromium-based browser or Safari, open `https://mail.yourdomain.com`
2. Look for the install icon in the address bar (or **Add to Home Screen** on mobile)
3. Click **Install** — the webmail now runs as a standalone app with desktop and mobile push notifications

# 6 · Disposable Email Aliases

DockFlare v3.1.1 added a disposable alias system enforced at the Cloudflare Worker layer. From the webmail Settings panel:

1. Click **Aliases → Create Alias**
2. Choose a generation style: `word-word-num`, `word-num`, `uuid-short`, or define a custom local-part
3. Optionally set an expiry date and a label/description
4. Save

Replies to an alias-received email pre-select that alias as the sender via a From dropdown. Expired aliases are deactivated hourly by a background job and their KV entries are removed from Cloudflare so unknown aliases reject at the SMTP edge.

| Limit | Value |
|-------|-------|
| Alias creations per hour | 20 |
| Total aliases per mailbox | 100 |
| Outbound rate (per sender) | 50/hr, 200/day |
{.dense}

# 7 · Backup and Restore

The Email section has its own backup tooling that captures both the SQLite mail database and the attachment volume.

- **Backup** → downloads a timestamped archive of the full mail store
- **Restore** → uploads an archive to recover from disaster
- **Teardown** → optionally wipes the local data **and** the Cloudflare-side R2 buckets, KV namespaces, and Workers for a complete uninstall

> 
> The mail volume at `/mnt/tank/configs/dockflare/mail` holds your **entire mail archive** — the SQLite database with FTS5 search index plus all message attachments. Make sure this path is on a ZFS pool covered by your snapshot/replication policy, and run a manual backup before any DockFlare image upgrade.
{.is-warning}

# 8 · Troubleshooting

| Symptom | Likely cause |
|---------|--------------|
| `401 Client Error: Unauthorized` on `/tokens/verify` during email setup | API token missing `User → User Details: Read`, or token's Account/Zone Resources are scoped to the wrong account — see section 1.3 |
| Email page shows red ❌ on R2 Storage | R2 is not activated on your Cloudflare account — see section 1.4 |
| `cloudflare-net` network not found at deploy | Run `docker network create cloudflare-net` before deploying the stack |
| `cloudflared` container never appears after wizard | DockFlare couldn't reach the Docker socket — check the `docker-socket-proxy` logs |
| `502 Bad Gateway` from `mail.yourdomain.com` | DockFlare's `cloudflared` container is running but not on `cloudflare-net` — restart the DockFlare master to recreate it |
| `mail.yourdomain.com` doesn't resolve | The tunnel hasn't picked up the webmail's labels yet — check the DockFlare dashboard for the rule, then wait a minute for DNS to propagate |
| Webmail loads but mailbox is empty | Mail manager hasn't pulled inbound messages from R2 yet — check `dockflare-mail-manager` logs |
| Outbound mail lands in spam | DKIM/SPF/DMARC not propagated yet — wait 5–10 minutes after one-click setup, then re-test |
| Permission denied on `/app/data` or `/data` | The `dockflare-init` container didn't run — confirm it completed and the host paths are owned by `65532:65532` |
{.dense}

# <img src="/youtube.png" class="tab-icon"> 9 · Video

https://youtu.be/PLACEHOLDER