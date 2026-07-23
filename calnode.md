---
title: Calnode
description: A guide to deploying Calnode
published: true
date: 2026-07-23T12:07:16.903Z
tags: 
editor: markdown
dateCreated: 2026-07-23T12:07:16.903Z
---

# What is Calnode?

**Calnode** is a lean, self-hostable Calendly alternative — a scheduling and booking engine that ships as a **single Go binary with an embedded SQLite database**. There is no Redis, no Postgres, no separate API server, and no multi-gigabyte image. 





# <img src="/docker.png" class="tab-icon"> 1 · Deploy Calnode


## 1.1 Generate your secrets first

Calnode encrypts stored credentials (SMTP passwords, OAuth tokens) at rest, so you need two random keys **before** you deploy:

```bash
openssl rand -hex 32   # CALNODE_ENCRYPTION_KEY
openssl rand -hex 32   # CALNODE_RECOVERY_SECRET
```

> **Save both of these somewhere safe and separate from the server.** Losing the encryption key makes your encrypted data unrecoverable unless you still have the recovery secret. Put them in Vaultwarden/Bitwarden before you go any further.
{.is-danger}

## 1.2 Compose file

```yaml
services:
  calnode:
    image: ghcr.io/calnode/calnode:latest
    container_name: calnode
    environment:
      - BASE_URL=https://booking.yourdomain.com
      - CALNODE_ENCRYPTION_KEY=paste_your_first_32_byte_hex_here
      - CALNODE_RECOVERY_SECRET=paste_your_second_32_byte_hex_here
      - DATABASE_URL=sqlite:///data/calnode.db
      - PORT=3000
      - LOG_LEVEL=info
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - /mnt/tank/configs/calnode:/data
```


Then browse to `https://booking.yourdomain.com` — it redirects to `/admin/` and walks you through first-run setup.

> **`BASE_URL` must include the scheme.** An `https://` prefix flips the app into production mode: secure cookies on, and the encryption key becomes mandatory. If you set `https://` and forget `CALNODE_ENCRYPTION_KEY`, the container refuses to start.
{.is-warning}



# 2 · Reverse proxy

Calnode does not terminate TLS — put Nginx Proxy Manager, Traefik, or Caddy in front of it.

There is one setting that trips people up:

> **Your proxy must forward the original `Host` header.** Calnode's CSRF protection compares the `Origin`/`Referer` header against `Host`. If your proxy rewrites Host, every admin write action returns a **403**. In Nginx Proxy Manager this is the default; in a hand-rolled nginx config, make sure you have `proxy_set_header Host $host;`.
{.is-danger}

Example nginx location block:

```nginx
location / {
    proxy_pass http://10.0.0.50:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

# 3 · Environment variables

| Variable | Required | Default | Notes |
|----------|----------|---------|-------|
| `BASE_URL` | Yes (prod) | `http://localhost:3000` | Identity host. Must include scheme. |
| `CALNODE_ENCRYPTION_KEY` | Yes when HTTPS | — | KEK input (Argon2id). App won't boot without it on https. |
| `CALNODE_RECOVERY_SECRET` | Recommended | — | Escrow secret so the data key survives a key rotation or loss. |
| `PUBLIC_BASE_URL` | No | = `BASE_URL` | Booker-facing host, if different from the admin host. |
| `DATABASE_URL` | No | `sqlite://./data/calnode.db` | Point at the persistent volume. |
| `PORT` | No | `3000` | The port the app listens on. |
| `EMAIL_SMTP_HOST` | No | — | Also settable in the admin UI. |
| `EMAIL_SMTP_PORT` | No | `587` | |
| `EMAIL_SMTP_USER` / `_PASS` | No | — | |
| `EMAIL_SMTP_STARTTLS` | No | `false` | On for port 587. |
| `EMAIL_SMTP_TLS` | No | `false` | On for port 465 (implicit TLS). |
| `EMAIL_FROM_ADDRESS` | No | `bookings@localhost` | |
| `EMAIL_FROM_NAME` | No | `Calnode` | |
| `GOOGLE_CLIENT_ID` / `_SECRET` | No | — | Google sign-in + calendar. |
| `LITESTREAM_REPLICA_URL` | Recommended | — | Setting this turns backups **on**. |
| `COOKIE_SECURE` | No | https→true | Override the cookie Secure flag. |
| `LOG_LEVEL` | No | `info` | `debug` / `info` / `warn` / `error` |


> Settings precedence is **environment variable > database setting > default**. If you set SMTP via env, you can't override it in the admin UI.
{.is-info}

# 4 · Configuration

## 4.1 First run

Open `https://booking.yourdomain.com/` → it redirects to `/admin/`. On a fresh database you'll be walked through creating the owner account. After that, work through:

1. **Settings → Email** — so confirmations actually send
2. **Settings → Google OAuth** — sign-in plus calendar free/busy
3. **Settings → Branding** — logo, business name
4. Create your first **event type** and set your **availability**

Migrations run automatically on every boot — there's no manual DB step.

## 4.2 Email

Gmail and Google Workspace SMTP will **rewrite your From address** to the authenticated account unless you've set up a verified "Send mail as" alias. For a branded From address, use a transactional provider. The project recommends **Resend**:

| Setting | Value |
|---------|-------|
| Host | `smtp.resend.com` |
| Port | `587` |
| Username | `resend` |
| Password | Your Resend API key |
| STARTTLS | On |
| From | `bookings@yourdomain.com` |


Verify your domain in Resend first (add the SPF/DKIM records it gives you — **DNS-only / grey cloud** if you're behind Cloudflare).

> Email settings are stored per-instance in that instance's database. If you run a staging copy alongside production, each one needs its own configuration.
{.is-info}

## 4.3 Google Calendar & OAuth

In Google Cloud Console → Credentials → your OAuth client, add these **Authorized redirect URIs**:

```
https://booking.yourdomain.com/v1/auth/callback
https://booking.yourdomain.com/v1/calendar/callback
```

Then set `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET`, or paste them into Settings → Google OAuth (that page shows you the exact redirect URIs for your running instance, which is handy).

> Google Calendar is a **sensitive scope**. Unverified apps get a warning screen and a 100-user cap. For a personal or small-team booking page that's usually fine — for anything public-facing, submit the app for verification.
{.is-warning}

Microsoft 365 / Outlook works the same way, and Calnode mints **Google Meet** or **Teams** links automatically when the host's connected calendar matches the platform.

## 4.4 Backups with Litestream

**Litestream is built into the image** and supervises the app process. It streams your SQLite file continuously to S3-compatible object storage with roughly a one-second RPO, and **restores automatically on boot** if the volume comes up empty.

> **Backups are OFF until you set the variables below.** A single volume with no replica is the biggest data-loss risk on this stack — turn this on before you take real bookings.
{.is-danger}

Five variables cover every S3-compatible provider:

```yaml
      - LITESTREAM_REPLICA_URL=s3://your-bucket/calnode
      - LITESTREAM_ENDPOINT=https://youraccountid.r2.cloudflarestorage.com
      - LITESTREAM_REGION=auto
      - LITESTREAM_ACCESS_KEY_ID=your_access_key
      - LITESTREAM_SECRET_ACCESS_KEY=your_secret_key
```

| Provider | `LITESTREAM_ENDPOINT` | `LITESTREAM_REGION` |
|----------|----------------------|---------------------|
| Cloudflare R2 | `https://<account-id>.r2.cloudflarestorage.com` | `auto` |
| Backblaze B2 | `https://s3.<region>.backblazeb2.com` | e.g. `us-west-004` |
| AWS S3 | *(leave unset)* | e.g. `us-east-1` |
| MinIO (self-hosted) | `https://minio.yourdomain.com` | `us-east-1` |


Two mistakes that catch everyone:

> - [x] `LITESTREAM_ENDPOINT` is the **account** endpoint — do **not** append the bucket name. R2 shows you the S3 API URL *with* the bucket in it; drop that last path segment.
> - [x] For R2, `LITESTREAM_REGION` is always **`auto`** — never the physical location (WNAM, ENAM, etc.). A real region string makes R2 reject the signature.
<!-- {blockquote:.is-warning} -->

If the endpoint or region are wrong, Litestream silently falls back to AWS and you'll see `InvalidAccessKeyId` (403) in the logs — that's your R2 key being sent to Amazon.

### Verifying the backup

Restore to a scratch path (**never** to `/data/calnode.db`) and check the file header:

```bash
docker exec -it calnode sh -lc '
  rm -f /tmp/check.db
  litestream restore -config /etc/litestream.yml -o /tmp/check.db /data/calnode.db
  ls -l /tmp/check.db
  head -c 16 /tmp/check.db; echo
  rm -f /tmp/check.db
'
```

If you see `SQLite format 3` and a file size close to your live database, the round-trip works. Run this drill when you first enable backups and periodically after — an unverified backup isn't a backup.

> The replica contains booking PII (names, email addresses, intake answers) in plaintext. **Keep the bucket private.** Calnode's own encrypted secrets stay sealed by the envelope key even inside the backup, but the booking data does not.
{.is-danger}

