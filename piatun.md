---
title: PIA-tun
description: A guide to deploying PIA-tun
published: true
date: 2026-05-30T11:10:56.041Z
tags: 
editor: markdown
dateCreated: 2026-05-30T11:10:56.041Z
---

#  What is pia-tun?

**pia-tun** is a feature-rich, security-focused VPN container for **Private Internet Access (PIA)** over **WireGuard**. It's a single Go binary on Alpine that connects to PIA, enforces a strict kill-switch, and lets other containers route all their traffic through the tunnel by sharing its network namespace. If you've used Gluetun, pia-tun is a similar idea but built specifically and exclusively around PIA — with PIA's dynamic registration, automatic port forwarding, and port syncing handled natively.


Highlights:

- **Strict kill-switch** — default-deny firewall (iptables-nft or legacy) that engages in ~25 ms on amd64 and stays up through startup, reconnects, crashes, and most misconfigurations
- **WireGuard speed** — measured at >95% of line speed with automatic MSS clamping
- **Automatic port forwarding** — acquires a PIA forwarded port, keeps it alive, and refreshes on expiry
- **Port syncing** — pushes the forwarded port straight into qBittorrent, Deluge, Transmission, or a custom endpoint with no scripting
- **SOCKS5 + HTTP proxies** — optionally let other machines/containers reach the VPN, with optional auth
- **Smart server selection** — latency-tests your chosen location(s) and picks the lowest-latency server
- **DoT support** — encrypt DNS to avoid leaks
- **No manual auth token** — the PIA token is fetched automatically and kept fresh
- **Observability** — `/health`, `/ready`, and Prometheus `/metrics` (plus a JSON variant)
- **Multi-arch images** — amd64, arm64, and armv7



# <img src="/docker.png" class="tab-icon"> 1 · Deploy pia-tun

```yaml
services:
  pia-tun:
    image: x0lie/pia-tun:latest
    container_name: pia-tun
    cap_add:
      - NET_ADMIN
    cap_drop:
      - ALL
    environment:
      - PIA_USER=your_pia_username
      - PIA_PASS=your_pia_password
      - PIA_LOCATIONS=ca_ontario,ca_toronto
      - PS_CLIENT=qbittorrent
      - LOCAL_NETWORKS=192.168.1.0/24
      - TZ=America/New_York
    ports:
      - "8080:8080"  # qBit WebUI — on pia-tun, not qbittorrent
    restart: unless-stopped

  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    network_mode: "service:pia-tun"
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/qbittorrent:/config
      - /mnt/tank/media/:/media
    depends_on:
      pia-tun:
        condition: service_healthy
    restart: unless-stopped
```

# 2 · Configuration

The full reference is in the project's [env docs](https://github.com/x0lie/pia-tun/blob/main/docs/env.md) — the most useful knobs:

## 2.1 VPN & Location

| Variable | Description | Default |
|----------|-------------|---------|
| `PIA_LOCATIONS` | Comma-separated regions (e.g. `ca_ontario,ca_toronto`); latency-tested, best server chosen | `all` |
| `PIA_DIP_TOKEN` | PIA Dedicated IP token (or secret `pia_dip_token`) | None |
| `WG_BACKEND` | `kernel` (faster) or `userspace` (wireguard-go); auto-detected | Auto |
| `TZ` | Timezone for log timestamps | None |
| `LOG_LEVEL` | `error`, `info`, `debug`, `trace` | `info` |


> 
> `PIA_LOCATIONS=all` works, but lowest latency doesn't always mean highest throughput. For most US users, PIA's Canadian servers (Ontario/Toronto) tend to give the best download speeds.
{.is-info}

## 2.2 Port Forwarding & Syncing

| Variable | Description | Default |
|----------|-------------|---------|
| `PF_ENABLED` | Enable PIA port forwarding (auto-on when `PS_CLIENT`/`PS_SCRIPT` set) | `false` |
| `PS_CLIENT` | `qbittorrent`, `transmission`, or `deluge` — syncs the forwarded port automatically | None |
| `PS_URL` | Client API endpoint; auto-set to `http://localhost:{8080,9091,8112}` per client | Auto |
| `PS_USER` / `PS_PASS` | Client credentials (or secrets `ps_user` / `ps_pass`) | None |
| `PS_SCRIPT` | Custom script run after each port refresh, for unsupported clients | None |
| `PORT_FILE` | File the forwarded port is written to | `/run/pia-tun/port` |


If your downloader has no usable API (e.g. rtorrent), pia-tun can't restart other containers itself (that would need the Docker socket). The project recommends a host-side `inotifywait` watcher on `PORT_FILE` that runs `docker restart`, or switching to an API-capable client. See their [dependent-restarts doc](https://github.com/x0lie/pia-tun/blob/main/docs/dependent-restarts.md).

## 2.3 Network, DNS & Kill-Switch

| Variable | Description | Default |
|----------|-------------|---------|
| `LOCAL_NETWORKS` | CIDRs allowed to bypass the tunnel (LAN access); `all`, `none`, or specific ranges | `auto` |
| `DNS` | `pia`, `system`, DoT (`tls://...`), or Do53 (`1.1.1.1`); round-robin | `pia` |
| `BOOTSTRAP_DNS` | Do53 only, used to resolve PIA endpoints; leave default | `149.112.112.9, .11` |
| `IPT_BACKEND` | `nft` or `legacy`; auto-detected | Auto |
| `MTU` | WireGuard interface MTU | `1420` |


The kill-switch is default-deny: only loopback, the VPN interface, and your container/local networks are allowed; everything else is dropped until it can go through the tunnel. It stays active during reconnects and even after a crash (until the network namespace is torn down). To access the WebUI from your LAN, set `LOCAL_NETWORKS` to your LAN CIDR as in the example above.

> 
> Do **not** point any dependent service's DNS at `149.112.112.9` or `149.112.112.11` on port 53 — pia-tun reserves those for bootstrap resolution of PIA endpoints, and overlap can cause problems. Also avoid `DNS=system` unless you fully understand your resolver chain; it's an easy way to leak DNS.
{.is-danger}

## 2.4 Proxies (optional)

To let *other* machines or containers use the tunnel without sharing the namespace, enable a proxy: `SOCKS5_ENABLED=true` (port `1080`) or `HTTP_PROXY_ENABLED=true` (port `8888`), optionally secured with `PROXY_USER`/`PROXY_PASS`. Remember to publish the relevant proxy port on the pia-tun container.

## 2.5 Observability

Metrics are on by default at `:9090` — `/health`, `/ready`, `/metrics` (Prometheus), and `/metrics?format=json`. Set `INSTANCE_NAME` to label metrics if you run more than one instance, and `METRICS_PORT` to move the listener. These are handy for a Grafana panel or an Uptime Kuma check against `/health`.

