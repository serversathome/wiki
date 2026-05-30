---
title: Rackpad
description: A guide to deploying Rackpad
published: true
date: 2026-05-30T13:24:08.606Z
tags: 
editor: markdown
dateCreated: 2026-05-30T13:24:08.606Z
---

#  What is Rackpad?

**Rackpad** is a self-hosted infrastructure inventory and operations app built specifically for homelabs and labs. If you've ever wanted NetBox's "single source of truth" for your gear but found NetBox heavier than your homelab needs, Rackpad is squarely aimed at that gap: rack and device inventory, port and cable tracking, IPAM, VLANs, WiFi, compute (hosts/VMs), network discovery, uptime monitoring, reports, and a topology visualizer — in one tidy single-container app.



What you can actually do with it:

- **Physical inventory** — racks with U-placement, mounted gear, plus room and loose-equipment context
- **Devices** — searchable, placement-aware records with parent/child relationships (VMs under hosts, wireless clients under APs)
- **Ports & cables** — switch/host/AP/VM/patch-panel connectivity, port templates, and cable mapping
- **IPAM** — subnets, DHCP scopes, IP zones, next-free allocation/release, and management-IP sync between device records and IPAM
- **VLANs** — individual VLANs and VLAN ranges
- **Compute** — virtualization hosts and VMs with CPU/memory/storage/spec capacity tracking
- **WiFi** — controllers, SSIDs, AP radios, and wireless-client telemetry (band, channel, signal, last-seen, roam context)
- **Discovery** — subnet scans with a review-first inbox, MAC/vendor enrichment, duplicate detection, and import into inventory
- **Monitoring** — per-device health checks with multiple targets per device (ICMP/TCP/HTTP/HTTPS), plus Email/Discord/Telegram alerts
- **Reports & Visualizer** — printable/PDF, Excel-compatible, and CSV exports, and rack/room/port/cable relationship maps
- **Hyper-V import** — a wizard that stages hosts and VMs from a local PowerShell export with editable mapping before import

It also includes optional OIDC SSO (authorization-code + PKCE), admin-only JSON backup export (which preserves password hashes but redacts stored alert secrets), an audit log, and sensible security headers (CSP, HSTS, X-Frame-Options) out of the box.


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Rackpad

```yaml
services:
  rackpad:
    image: ghcr.io/kobii-git/rackpad:1.3.0
    container_name: rackpad
    init: true
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
      HOST: 0.0.0.0
      PORT: 3000
      DATABASE_PATH: /data/rackpad.db
      MONITOR_INTERVAL_MS: 300000
      # Reverse-proxy hardening — see section 2.3:
      TRUST_PROXY: 0
      TRUSTED_HOSTS: ""
      TRUSTED_ORIGINS: ""
      APP_URL: ""
      # Optional OIDC SSO — see section 2.4:
      OIDC_ENABLED: 0
      OUI_AUTO_UPDATE: 1
      DISCOVERY_MAC_SCAN_MODE: auto
    volumes:
      - /mnt/tank/configs/rackpad:/data
    read_only: true
    tmpfs:
      - /tmp
    security_opt:
      - no-new-privileges:true
    healthcheck:
      test: ["CMD", "node", "-e", "fetch('http://127.0.0.1:3000/api/health').then(r=>process.exit(r.ok?0:1)).catch(()=>process.exit(1))"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 10s
    restart: unless-stopped
```

Open `http://<your-host>:3000` and create the initial admin account — on first boot there are no users, so the first account you make becomes the admin. You'll then be asked whether to start empty or preload the expanded demo environment (handy for a first look).


# 2 · Configuration

## 2.1 First Run

1. Browse to `http://<your-host>:3000`
2. Create the initial admin account and sign in
3. Choose **start empty** or **preload demo data**
4. Begin documenting racks, devices, VLANs, and IPAM

User roles are admin/editor/viewer, so you can give read-only access to others without handing over edit rights.

## 2.2 Discovery & MAC/Vendor Enrichment

Subnet discovery scans and imports into inventory work normally, but **MAC/vendor enrichment needs layer-2 (ARP) visibility from the container**. On standard Docker bridge networking, MAC details may be unavailable — the project notes this can happen on bridge networking, routed VLANs, VPNs, or containers without raw-socket capability, and Rackpad will surface scan diagnostics when it does. `DISCOVERY_MAC_SCAN_MODE=auto` tries `arp-scan`/`nmap` and falls back to the OS neighbor cache. If you specifically want full MAC/vendor enrichment, run the container with host networking or the needed capabilities so it shares the LAN's L2 segment — at the cost of the read-only/bridge isolation in the default compose. For most people, bridge networking plus occasional manual entry is fine.


## 2.3 OIDC SSO (optional)

Rackpad supports OIDC via the authorization-code flow with PKCE. Set `OIDC_ENABLED=1` and the issuer/client values, and point your provider's redirect URI at `APP_URL/api/auth/oidc/callback` (or set `OIDC_REDIRECT_URI` explicitly behind a proxy). An Authentik example from the docs:

```yaml
      OIDC_ENABLED: 1
      OIDC_ISSUER_URL: https://authentik.serversatho.me/application/o/rackpad
      OIDC_CLIENT_ID: your-client-id
      OIDC_CLIENT_SECRET: your-client-secret
      OIDC_REDIRECT_URI: https://rackpad.serversatho.me/api/auth/oidc/callback
      OIDC_LABEL: Authentik
      OIDC_DEFAULT_ROLE: viewer
      OIDC_ADMIN_GROUPS: admin
```

> 
> `OIDC_ISSUER_URL` must be the provider **issuer**, not the authorize URL or client settings page — Rackpad fetches `OIDC_ISSUER_URL/.well-known/openid-configuration`. For per-application issuers (like Authentik), use the application/provider issuer path, not the IdP root domain. If sign-in returns a 502/404, test that discovery URL directly, or set `OIDC_DEBUG=1` temporarily to log the discovery URL, redirect URI, token-endpoint status, and JWKS URL.
{.is-info}

