---
title: elm.chat
description: A guide to deploying elm.chat
published: true
date: 2026-08-28T20:24:30.358Z
tags: 
editor: markdown
dateCreated: 2026-08-28T20:24:30.358Z
---

# What is elm.chat?

**elm.chat** is a disposable, end-to-end encrypted chat application built entirely on Cloudflare Workers and Durable Objects. Rooms are created instantly with no accounts, no signup, and no server-side transcript — messages and files are encrypted in your browser, relayed through the edge as ciphertext, and the room self-destructs on a timer you set.

The room secret lives in the URL fragment (`#secret`), which browsers never send to the server. That means the operator of an instance — including you, if you self-host it — cannot read anything that passes through it.

> **This is not a Docker container.** elm.chat is Cloudflare-native and has no container image and no TrueNAS app. It runs as a single Worker plus one Durable Object per room. There is no way to deploy it on your NAS or a Docker host — the Durable Object runtime only exists on Cloudflare's edge.
{.is-warning}



---

# <img src="/cloudflare.png" class="tab-icon"> 1 · Deploy elm.chat

# {.tabset}
## Wrangler CLI (recommended)

This is the fully tested path. You need **Node.js 18+**, **npm**, **Git**, and a free Cloudflare account.

```bash
# 1. Clone and install
git clone https://github.com/shawnbure/elm-chat.git
cd elm-chat
npm install

# 2. Authenticate Wrangler (opens a browser)
npx wrangler login

# 3. Only if your Cloudflare login has more than one account
export ELM_CHAT_CLOUDFLARE_ACCOUNT_ID=<your-account-id>

# 4. Build the web app and deploy the Worker + Durable Object
npm run deploy
```

`npm run deploy` builds `apps/web/dist` and then deploys the Worker in one step. On success Wrangler prints your live URL, something like `https://elm-chat.<your-subdomain>.workers.dev`.

Open it, click **Create private conversation**, and you have a working room.

> **First deploy to a new account?** Cloudflare may ask you to register a free `*.workers.dev` subdomain under **Workers & Pages** in the dashboard. Do that once, then re-run `npm run deploy`.
{.is-info}

> The `migrations` block in `workers/api/wrangler.jsonc` creates the `RoomDurableObject` SQLite class automatically on first deploy. There is no manual migration step and no database to provision.
{.is-success}

## One-Click Deploy

1. Click the **Deploy to Cloudflare** button on the [project README](https://github.com/shawnbure/elm-chat).
2. Authorize Cloudflare to fork the repo into your GitHub account and connect it to Workers Builds.
3. When prompted for build settings:
   - **Build command**: `npm install && npm run build`
   - **Wrangler config path**: `workers/api/wrangler.jsonc` (leave as-is)
4. Deploy. Cloudflare provisions the Worker and Durable Object namespace for you.

> The one-click path builds on Cloudflare's infrastructure and occasionally fails on build settings. If it does, fall back to the Wrangler CLI tab — that's the path the maintainer actively tests.
{.is-warning}



# 2 · Configuration

## 2.1 Custom Domain

To serve the app from your own domain instead of `*.workers.dev`:

1. Add the domain to your Cloudflare account — it must use Cloudflare DNS.
2. Go to **Workers & Pages → your Worker → Settings → Domains & Routes → Add custom domain**.

Alternatively, add a `routes` entry to `workers/api/wrangler.jsonc` and redeploy. Cloudflare provisions the TLS certificate automatically either way.

## 2.2 Abuse Protection with Turnstile

Room creation can be gated behind an invisible [Cloudflare Turnstile](https://developers.cloudflare.com/turnstile/) challenge. It's completely inert until you configure keys, so everything above works without it.

1. In the Cloudflare dashboard, create a Turnstile widget (Managed or Invisible) for your domain. You get a **site key** (public) and a **secret key** (private).
2. Give the web build the site key:

```bash
VITE_TURNSTILE_SITE_KEY=<site-key> npm run build
```

Or add it to a `.env` file under `apps/web`.

3. Give the Worker the secret key:

```bash
cd workers/api
npx wrangler secret put TURNSTILE_SECRET
```

With both configured, the landing page runs an invisible challenge before creating a room and the Worker rejects room creation unless the token verifies. With neither set, room creation is open to anyone who can reach your instance.

> If you're publishing your instance URL anywhere public, turn Turnstile on. Open room creation on a free-tier Worker is an easy way to burn through your daily request budget.
{.is-warning}

## 2.3 Room Policy Options

These are set per-room by the creator on the landing page, not in config:

| Setting | Options |
|---------|---------|
| Message vanish | minutes / hours / days / indefinite |
| Room self-destruct | minutes / hours / days / indefinite |
| Invites | single-use, revocable |
| Participant removal | creator-only |


The creator token is stored in the browser's `localStorage`. Clear it and you lose creator control of that room.

## 2.4 Renaming the Worker

To run multiple instances, or to avoid a name clash on your account, change the `"name"` field in `workers/api/wrangler.jsonc` before deploying.



# 3 · Local Development

The app runs as two processes in development:

- The Cloudflare Worker + Durable Object under `wrangler dev` on `http://localhost:8799`
- The Vite dev server (React, hot reload) on `http://localhost:3000`

Vite proxies `/api` — including the room WebSocket — to the Worker, so behavior matches production.

```bash
npm install
npm run build     # once, creates apps/web/dist which wrangler dev expects
npm run dev       # starts both processes
```

Open `http://localhost:3000`, create a room, then open the copied link in a second browser or incognito window to see live encrypted chat between two participants.

> Port `8799` is deliberate — it avoids colliding with other Cloudflare projects that default to `8787`.
{.is-info}



# 4 · How It Works

## 4.1 Message Flow

1. The landing page generates a random 256-bit `room_secret` **in your browser**.
2. `POST /api/rooms` creates the room metadata and its Durable Object.
3. The browser navigates to `/c/:roomId#<room_secret>` — the fragment never leaves the client.
4. The client derives the room key locally from that fragment using HKDF-SHA-256.
5. One WebSocket carries join, presence, encrypted chat, encrypted file chunks, transcript sync, kicks, destroy, and keepalives.
6. The Durable Object relays each encrypted envelope — broadcast to everyone else, or targeted to a single session for file chunks and per-peer sync.
7. New joiners request transcript sync from connected peers, who replay their local encrypted history.

## 4.2 Where Data Lives

| Location | What's there |
|----------|--------------|
| Durable Object storage (persisted) | Room metadata, creator token, one-time invites |
| Relayed, never persisted | Encrypted chat envelopes, encrypted file chunks, transcript sync payloads |
| Never visible to the server | Plaintext of any kind, the room secret, participants' IP addresses |
| Client memory | The encrypted transcript, held by each connected participant |


If no connected peer still holds a message or file, it is gone. There is no archive to recover.

## 4.3 Why Relay Instead of Peer-to-Peer

elm.chat deliberately does **not** use WebRTC and contacts no STUN or TURN servers. Relaying through the Durable Object:

- Keeps every participant's IP address private from other room members — WebRTC would leak peer IPs via ICE candidates
- Needs no TURN server
- Works reliably on mobile and restrictive networks where direct P2P fails on symmetric NAT

The trade-off: an honest-but-curious server relays ciphertext and can observe connection metadata — timing, sizes, presence.

## 4.4 Crypto Details

| Component | Implementation |
|-----------|----------------|
| Room key | HKDF-SHA-256 over the fragment secret → AES-GCM-256 |
| Messages | AES-GCM, random 96-bit nonce per message, base64url envelope |
| Files | 64 KiB chunks, each AES-GCM encrypted with its own nonce, 25 MiB max |
| Identity keys | Ephemeral ECDSA P-256 per client, shared on join (reserved for future message authentication) |




# 5 · Security Notes

> **Read this before you recommend elm.chat to anyone with a real threat model.** The maintainer's own disclaimer is explicit: do not market or rely on this project as a completed high-assurance safety tool until its protocol, implementation, and operational guarantees have been independently reviewed and tested under realistic conditions.
{.is-danger}

Encryption is not the whole picture. What single-use invites and E2EE do **not** solve:

- An invite intercepted before the intended recipient redeems it still admits the first redeemer
- A compromised device leaks via screenshots, clipboard history, browser history, or malware
- A participant who forwards plaintext, screenshots, or the room secret after joining defeats the protocol entirely
- Transport and endpoint metadata exists even when content is encrypted
- Traffic analysis against the relay — message timing and size are observable
- A room left open too long grows the exposure window regardless of invite policy

Operational best practices:

> **Room Hygiene**
> - [x] Issue invites only when the recipient is ready to use them
> - [x] Keep invite lifetime short
> - [x] Revoke unused invites promptly
> - [x] Remove participants when they no longer need access
> - [x] Keep message expiry and self-destruct aggressive
> - [x] Destroy the room as soon as the conversation is done
> - [x] Treat every endpoint as a possible weak point



