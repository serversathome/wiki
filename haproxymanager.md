---
title: HA Proxy Manager
description: A guide to deploying HA Proxy Manager
published: true
date: 2026-08-28T20:44:54.637Z
tags: 
editor: markdown
dateCreated: 2026-08-28T20:44:54.637Z
---

# What is HAProxy Cluster Manager?

**HAProxy Cluster Manager** (`haproxy-manager`, or *HAM*) is a small self-hosted web UI that manages an HAProxy configuration for you, requests and renews Let's Encrypt certificates through `acme.sh`, and ties any number of nodes together with Keepalived on a shared virtual IP. One node holds the VIP and serves traffic, the rest stand by with the same configuration and take over if it disappears.

If you have ever configured HAProxy inside OPNsense, the layout will look familiar — Public Services, Backend Pools, Real Servers, Conditions, Rules and Health Monitors are all here. The difference is that this runs on its own box (or in a container) instead of on your firewall, and it writes a plain `haproxy.cfg` and `keepalived.conf` rather than hiding the result from you. There is no database; all state lives in a single `config.json`.


> HAProxy Cluster Manager expects to own `/etc/haproxy/haproxy.cfg`. The first **Apply** overwrites whatever is there (keeping a `.bak`). Do not point this at an existing hand-tuned HAProxy install you care about.
{.is-warning}

# 1 · Deploy HAProxy Cluster Manager
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

```yaml
services:
  haproxy-manager:
    image: ghcr.io/avandeputte/haproxy-manager:latest
    container_name: haproxy-manager
    restart: unless-stopped
    network_mode: host
    cap_add:
      - NET_ADMIN
      - NET_BROADCAST
      - NET_RAW
    environment:
      HAM_LISTEN: "0.0.0.0"
      HAM_PORT: "8080"
    volumes:
      - /mnt/tank/configs/haproxy-manager/data:/var/lib/haproxy-manager
      - /mnt/tank/configs/haproxy-manager/acme:/var/lib/acme.sh
      - /mnt/tank/configs/haproxy-manager/haproxy:/etc/haproxy
      - /mnt/tank/configs/haproxy-manager/keepalived:/etc/keepalived
```

1. Deploy the stack, then browse to `http://<node-ip>:8080`
2. Create the administrator account when the UI asks for one
3. Run one container per node — the image is all-in-one (manager, HAProxy, Keepalived and `acme.sh` under `supervisord`)

Images are published for **linux/amd64** and **linux/arm64**. Pin a tag such as `:1.46` in production; `:latest` moves whenever `main` does.

> Host networking is the intended mode. Keepalived's VRRP and the virtual IP need a real interface, and HAProxy binds whatever ports your Public Services define. Bridge mode works for a single standalone node with no VIP, but then you have to publish every port yourself — `8080` for the UI, `80`, `443`, and `9080` for the HTTP-01 listener.
{.is-info}

> There is no systemd inside a container, so `supervisord` restarts the app only if it *exits*. A **hung** manager will not be restarted — the image's `HEALTHCHECK` reports it, but something has to act on that. For a production cluster the native install is the better fit.
{.is-warning}

## <img src="/ubuntu.png" class="tab-icon"> Debian / Ubuntu

Run this on **every** node (Debian 12/13, Ubuntu 22.04/24.04):

```bash
curl -fsSL https://raw.githubusercontent.com/avandeputte/haproxy-manager/main/install.sh | sudo bash
```

Or grab the `.deb` / `.rpm` attached to any release:

```bash
sudo apt-get install -y ./haproxy-manager_1.91.1_all.deb
```

The installer pulls in `haproxy`, `keepalived`, `python3-flask`, `python3-requests`, `python3-waitress`, `openssl`, `socat` and `iproute2`, drops in a pinned `acme.sh` with its own cron disabled, enables `net.ipv4.ip_nonlocal_bind` so HAProxy can bind a VIP this node does not currently hold, generates the administrator login and API key, and installs the systemd unit.

The generated password is printed at the end and also written to `/var/lib/haproxy-manager/admin-credentials.txt` (mode 0600).

> Re-running the same command on an existing install offers **update**, **remove** (keeps `config.json` and certificates), **purge** or cancel. Piped from `curl` with no terminal to prompt on, it updates in place.
{.is-info}

> The service runs as **root**, because it writes to `/etc/haproxy` and `/etc/keepalived` and reloads system services. Restrict who can reach port 8080.
{.is-danger}

# 2 · First Run

## 2.1 Create the admin and pick a cluster mode

The first visit asks for a username and password, then offers a setup wizard with two branches:

| Branch | What it does |
|--------|--------------|
| Join an existing cluster | Point it at a node already running and paste that node's API key. It registers itself and the other node pushes the whole configuration, cluster settings and membership list back. You don't touch the other nodes. |
| Create a new cluster / standalone | Set the virtual IP the nodes will share, or skip it and run alone. Other nodes join later by pointing at this one. |


Either path ends by showing **this node's API key** — that is what the other nodes need to reach it. Save it somewhere.


# 3 · Publishing a Service

**Services → Publish a service**. Two lines get you a working HTTPS reverse proxy:

```
Public URL    https://jellyfin.example.com
Forward to    http://192.168.1.100:8096
```

From those the wizard creates the Real Server, Backend Pool, host Condition, routing Rule, HTTPS listener, ACME certificate and an HTTP listener that redirects to HTTPS — then applies. Hit **Preview** first to see the objects and the resulting `haproxy.cfg`.

## 3.1 Things worth knowing

- **Wildcards are reused, not duplicated.** If `*.example.com` already covers `app.example.com`, the wizard attaches the existing certificate instead of burning a new issuance. An exact name beats a matching wildcard, and `*.example.com` does *not* cover `example.com` or `a.b.example.com`.
- **Several public URLs** on one service: one per line, all reaching the same pool through a single rule. They must agree on scheme and port, since they share a listener.
- **Several targets**, comma separated, are load balanced. Each can be named: `galera1=192.168.1.81:3306`.
- **Raw TCP** works — enter `tcp://0.0.0.0:3306` as the public URL and you get a TCP listener, a TCP-mode pool and the servers. TCP carries no host name, so one port serves exactly one pool.
- **Paths** route too: `https://app.example.com/api` matches only that prefix, and more specific rules are placed ahead of broader ones.
- **Publishing the same URL again edits it** rather than quietly stacking a second server behind it.
- **Pause** puts a service into maintenance: every request gets a clean 503 while the servers and their health checks stay exactly as they were.

## 3.2 Health checks

Set the check in the same step as the service; servers that fail it drop out of rotation.

| Check | What HAProxy does |
|-------|-------------------|
| ping | Opens a TCP connection to the port (HAProxy has no ICMP ping) |
| HTTP request | `option httpchk` with a path and expected status |
| TLS handshake | `option ssl-hello-chk` |
| PostgreSQL login | `option pgsql-check` — login handshake only, no password |
| MariaDB / MySQL login | `option mysql-check`, `post-41` for anything modern |


The check can run on a **different port and protocol** from the traffic — route TCP to 5432 while an HTTP check asks each node's Patroni API on 8008 whether it is the primary.

## 3.3 Requiring a sign-in

Both methods are enforced by HAProxy itself, so an unauthenticated request never reaches your backend:

- **Basic authentication** — users and groups managed under **Sign-in**, checked against a `userlist` in the generated config. Passwords stored as SHA-512 crypt hashes. A service admits *groups*, so access changes by moving people in or out of one.
- **Single sign-on (OIDC)** — send visitors through Authentik, Keycloak, Authelia, Pocket ID, Google or Entra, with a per-service allow-list of emails and `@domains`. HAProxy verifies the session on every request using an HMAC-signed cookie in pure configuration — no Lua, HAProxy 2.4+ — so a failover signs nobody out.

**Allowed networks** (a CIDR allow-list, which also works for `tcp://` services) and **Skip the sign-in from** (trust the LAN) compose with either.

# 4 · Certificates

**Certificates → Request a certificate** asks for the domains, then lets you reuse or create the ACME account and the challenge type in the same step.

- The **DNS API hook** list is parsed out of the `acme.sh` on the node — 191 providers — and picking one shows exactly which credential variables it needs.
- **Preview** names every object that will be created or reused before anything is saved.
- It warns about the mistakes you'd otherwise only find in a failed issuance: a wildcard with an HTTP-01 challenge, a DNS-01 challenge with no API hook, an account with no email address.
- Before a real certificate exists, Apply drops in a short-lived **self-signed placeholder** so HAProxy can start. The first successful issue replaces it.

> Use the `letsencrypt_test` CA while you are setting things up. It issues untrusted certificates with no rate limits, which is exactly what you want before you've got the DNS hook right.
{.is-success}

> **Only the node holding the virtual IP issues and renews.** HTTP-01 validation arrives at that address, and with DNS-01 the nodes would race each other and burn the CA's rate limits. A passive node says so and refuses a manual issue.
{.is-info}

Once a certificate is written, HAProxy is reloaded so it actually serves it, and the PEM is pushed to every other node. Nothing to configure.

# 5 · High Availability

## 5.1 What syncs and what doesn't

| Synced between nodes | Node-local (never synced) |
|----------------------|---------------------------|
| Real Servers, Backend Pools, Public Services | Keepalived (interface, VRID, priority, VIP) |
| Conditions, Rules, Health Monitors | The peer list and their API keys |
| HAProxy Settings | This node's API key |
| ACME accounts, challenges, certificates | Administrator login |
| Deployed certificate PEM files | |


Sync is push-based: the node you edit pushes to the others, and a renewed certificate is pushed by whichever node renewed it.

## 5.2 Only the active node is editable

The node holding the VIP is where the shared configuration is edited. The others are read-only and say so in a banner, which stops nodes from diverging and then overwriting each other. A passive node can still fix *itself* — interface, priority, peer list, login, API key, updates and Apply.

> If **no** node holds the VIP, the banner's **Edit here anyway** unlocks that node, so a broken cluster is never a lockout. The unlock belongs to that sign-in and is not stored.
{.is-info}

## 5.3 Revisions keep the nodes honest

The shared configuration carries a revision counter and a fingerprint. Every node reports both, and the Cluster table says **configuration agreed** or **configuration differs** at a glance. A node that is behind gets named, a node cannot push a configuration older than the one already there, and with *Keep the nodes in step* on, stale nodes are refreshed automatically from whichever node holds the newest.

## 5.4 Keepalived notes

- Keepalived runs on every node that has a virtual IP configured — it is not a per-node switch.
- Unicast addresses are derived from the node list automatically. Nothing to type.
- All nodes must agree on the **virtual router ID**. For non-preempting failover, set every node to `BACKUP` with a different priority and enable `nopreempt`.
- **Tracking HAProxy** through its admin socket (the default) beats the `process` option: a wedged HAProxy that accepts connections and answers none still has a process, but it will fail the socket check and hand the VIP to a node that works.

> Bind your Public Services to the virtual IP, not to a node's own address.
{.is-warning}

# 6 · Monitoring

## 6.1 Watchdog

Each node supervises its own services and restarts them deliberately, not reflexively: never against a configuration `haproxy -c` rejects, never a unit you masked or disabled, and never more than three times per fifteen minutes.

Once a minute the node holding the VIP also **requests every published URL the way a browser would** — resolve the name, connect, speak TLS, ask. That catches the failures every other layer is blind to: DNS pointing at the wrong machine, another host claiming the address, a listener that lost its certificate.

It also runs `arping -D` every few minutes to check whether anything else on the network answers for the addresses this node holds. Two machines claiming one address looks like almost anything except what it is.

## 6.2 Notifications

Email (SMTP), Pushover, and a generic webhook that posts `{subject, message, severity, event, node, time}`. Test each destination from the page — it sends a real message.

Alerts fire on **transitions**, not conditions, and an unresolved problem repeats only every 6 hours. Each service carries an **Alert when** setting: *a server is lost* (default), *no server is left* (correct for leader-election pools like Patroni, where one healthy server is the design), or *never*.

> Notification settings are shared between nodes, which means SMTP passwords and Pushover tokens travel in the sync payload. Run peer sync over HTTPS or keep it on a trusted network.
{.is-warning}

## 6.3 Home Assistant and Prometheus

Point **Notifications → Home Assistant** at an MQTT broker and the entities appear on their own via MQTT discovery — a problem sensor per service, a connectivity sensor per published URL, days-to-expiry per certificate, configuration drift, and requests per minute. On failover the new active node continues publishing the same topics, so entities carry on instead of duplicating.

`GET /metrics` speaks the Prometheus exposition format and requires the node's API key:

```yaml
scrape_configs:
  - job_name: haproxy-manager
    authorization:
      credentials: <the API key from Cluster - This node>
    static_configs:
      - targets: ["proxy1:8080", "proxy2:8080", "proxy3:8080"]
```

Scrape every node; `ham_node_active` tells you which one holds the VIP.

# 7 · Recovery

`config.json` is written by rename so it is never half-written, and the previous copy is kept as `config.json.bak`. `haproxy.cfg` and `keepalived.conf` get a `.bak` before each Apply.

The management UI is served by the app itself on port 8080 — **not** through HAProxy — so `http://<node-ip>:8080` still reaches it when HAProxy is misrouting. To roll HAProxy back:

```bash
cp /etc/haproxy/haproxy.cfg.bak /etc/haproxy/haproxy.cfg
systemctl reload haproxy
```

**Settings → History** keeps the last 50 states of the shared configuration on each node, with a diff against now and a **Restore** that comes back as a *new* revision so the rest of the cluster accepts it. Nothing is applied or synced until you press Apply.

**Settings → Backup & Export** downloads the rendered `haproxy.cfg` and `keepalived.conf`, or a JSON configuration backup. The backup deliberately contains **no secrets** — no API key, no login, no private keys, no DNS credentials — which makes it safe to keep off the node.

# 8 · Environment Overrides

| Variable | Default | Notes |
|----------|---------|-------|
| `HAM_LISTEN` | `0.0.0.0` | Set to `127.0.0.1` once the UI is published through HAProxy |
| `HAM_PORT` | `8080` | UI and API |
| `HAM_THREADS` | `16` | Waitress worker threads |
| `HAM_PEER_CONNECT_TIMEOUT` | `3` | How long to wait for a node to accept a connection |
| `HAM_PEER_READ_TIMEOUT` | `5` | How long to wait for a health query answer |
| `HAM_PUSH_READ_TIMEOUT` | `90` | How long to wait for a node to accept a pushed config |
| `HAM_CLUSTER_POLL` | `15` | Background cluster health collection interval |
| `HAM_CLUSTER_MAX_AGE` | `60` | Age at which a stale snapshot is collected inline |
| `HAM_DATA_DIR` | — | Where `config.json` lives |
| `HAM_DEBUG` | — | `1` for verbose logging |
| `HAM_DRY_RUN` | — | `1` skips `systemctl` calls; render and validate only |


> **Address peers by IP, not by name.** DNS resolution happens *before* any of those timeouts start counting, and a stock Debian or Ubuntu box caches nothing — a nameserver that doesn't answer costs ten seconds per lookup, every time. It is also the wrong dependency: the name may be published by the very cluster that is in trouble.
{.is-warning}

> Do not put a multi-process WSGI server in front of this. The app keeps state in process globals — the write lock, the failed sign-in counters, the renewal timer — so several worker processes would each get their own copy and let concurrent edits overwrite one another. Raise `HAM_THREADS` instead.
{.is-danger}

