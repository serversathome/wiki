---
title: Home
description: This is the official wiki for Servers@Home
published: true
date: '2026-11-04T15:55:40.951Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:05:13.120Z'
---

Your guide to self-hosting, homelabs, and taking control of your data. Tested guides, open source solutions, privacy-respecting infrastructure.

*247 guides · maintained by Evan and contributors like you*

> [!NOTE]
> Use the sidebar or the search bar at the top to find guides. On mobile, tap the blue menu icon ☰ at the bottom-left.

# Start here

::block-links-grid
```yaml
- title: "🚀 First-time homelab guide"
  description: "Where to begin and what to buy"
  url: "https://blog.serversatho.me/breaking-the-rules/"
  color: amber
- title: "🗺️ Overview map"
  description: "What every piece of a media server actually does"
  url: "/OverviewMap"
  color: amber
- title: "TrueNAS"
  description: "Install and run your storage OS"
  url: "/TrueNAS"
  icon: "img:/_files/truenas-core.png"
  color: amber
- title: "Docker"
  description: "Run your first containers"
  url: "/Docker"
  icon: "img:/_files/docker.png"
  color: amber
- title: "Dockge"
  description: "Manage compose stacks from a browser"
  url: "/Dockge"
  icon: "img:/_files/dockge.png"
  color: amber
```
::

> [!NOTE]
> Stuck on something specific? Try the [FAQ](/faq) first, then ask in [Discord](https://discord.gg/syvCPcRJnR).

# Browse by category

::block-links-grid
```yaml
- title: "TrueNAS"
  description: "Pools, datasets, permissions, shares, apps and hardening"
  url: "/TrueNAS"
  icon: "img:/_files/truenas-core.png"
  color: purple
- title: "🖥️ Server management"
  description: "Proxmox, backups, monitoring, dashboards, game servers"
  url: "/servermanagement"
  color: purple
- title: "🎬 Media management"
  description: "Media servers, photos, music, books, games"
  url: "/mediamanagement"
  color: purple
- title: "📺 *Arr stack"
  description: "Radarr, Sonarr, Prowlarr, download clients, quality control"
  url: "/arr-stack"
  color: purple
- title: "🌐 Networking"
  description: "VPNs, reverse proxies, ad blocking, DNS"
  url: "/Networking"
  color: purple
- title: "Containerization"
  description: "Docker, Dockge, Portainer, updates, logging"
  url: "/containerization"
  icon: "img:/_files/docker.png"
  color: purple
- title: "🧰 Productivity apps"
  description: "Notes, files, passwords, money, automation, self-hosted AI"
  url: "/productivity"
  color: purple
- title: "🌍 Web hosting"
  description: "Ghost, WordPress, analytics, wikis, VPS"
  url: "/webhosting"
  color: purple
```
::

Building a media server? Read [install instructions](/InstallInstructions) and [folder structure](/Folder-Structure) before you deploy anything. Getting hardlinks right the first time saves a rebuild later.

# Popular guides

::block-links-grid
```yaml
- title: "Jellyfin"
  description: "Volunteer-built, fully free media server"
  url: "/jellyfin"
  icon: "img:/_files/jellyfin.png"
  color: lime
- title: "Immich"
  description: "Photo and video management with mobile backup"
  url: "/immich"
  icon: "img:/_files/immich.png"
  color: lime
- title: "Nextcloud"
  description: "Google Workspace replacement"
  url: "/nextcloud"
  icon: "img:/_files/nextcloud.png"
  color: lime
- title: "wg-easy"
  description: "WireGuard VPN with a friendly GUI"
  url: "/wg-easy"
  icon: "img:/_files/wireguard.png"
  color: lime
- title: "Cloudflare Tunnels"
  description: "Expose services without opening ports"
  url: "/CloudflareTunnels"
  icon: "img:/_files/cloudflare.png"
  color: lime
- title: "NetBird"
  description: "Zero-config mesh VPN"
  url: "/netbird"
  icon: "img:/_files/netbird.png"
  color: lime
- title: "Vaultwarden"
  description: "Bitwarden-compatible password manager"
  url: "/vaultwarden"
  icon: "img:/_files/vaultwarden.png"
  color: lime
- title: "qBittorrent"
  description: "Torrent client, behind Gluetun"
  url: "/qBittorrent"
  icon: "img:/_files/qbittorrent.png"
  color: lime
```
::

# Watch the series

Most guides here have a video. If you would rather be walked through it:

::block-links-grid
```yaml
- title: "▶️ TrueNAS basics"
  description: "Your first NAS, start to finish"
  url: "https://www.youtube.com/playlist?list=PL6zQmF2gDqDT7SHyBe7ni1P2S4NzyJpD6"
  color: blue
- title: "▶️ TrueNAS and Docker guides"
  description: "Deploying the apps documented on this wiki"
  url: "https://www.youtube.com/playlist?list=PL6zQmF2gDqDTOLfGfYN9lIYw5WkXXZ5yu"
  color: blue
- title: "▶️ Proxmox series"
  description: "Virtualization instead of, or alongside, TrueNAS"
  url: "https://www.youtube.com/playlist?list=PL6zQmF2gDqDRMv3Mc-vYLnMzxUXEOd6Ns"
  color: blue
- title: "▶️ Top 10 new applications"
  description: "What is worth self-hosting this month"
  url: "https://www.youtube.com/playlist?list=PL6zQmF2gDqDQvUKdCaARpOYqNsbTVpWpE"
  color: blue
- title: "▶️ State of self-hosting"
  description: "Where the ecosystem is heading"
  url: "https://www.youtube.com/playlist?list=PL6zQmF2gDqDSxiqFBQkGjnKBrvOcCnEHt"
  color: blue
```
::

> [!TIP]
> Want it structured? [TrueNAS Fundamentals for Beginners](https://www.udemy.com/course/truenas-fundamentals-for-beginners/?referralCode=8369F4A322AA6867F03C) on Udemy covers TrueNAS from the ground up in one course.

# From the blog

The wiki tells you how. The [blog](https://blog.serversatho.me/) argues about what and why.

- [Streaming media is officially ridiculous](https://blog.serversatho.me/streaming-media-is-officially-ridiculous/)
- [Getting SMART about your data: why drive health isn't enough](https://blog.serversatho.me/getting-smart-about-your-data-why-drive-health-isnt-enough/)
- [Your self-hosted office suite is going through some things right now](https://blog.serversatho.me/your-self-hosted-office-suite-is-going-through-some-things-right-now/)
- [Sourcing used parts](https://blog.serversatho.me/sourcing-used-parts/), on buying secondhand hardware without getting burned
- [Best.VPN.Ever.](https://blog.serversatho.me/best-vpn-ever/), AirVPN plus WireGuard for your homelab

# Connect and contribute

::block-links-grid
```yaml
- title: "Discord"
  description: "Real-time help, fastest response for technical questions"
  url: "https://discord.gg/syvCPcRJnR"
  icon: "img:/_files/discord.png"
  color: teal
- title: "YouTube"
  description: "Video walkthroughs of most guides here"
  url: "https://youtube.com/@Serversathome"
  icon: "img:/_files/youtube.png"
  color: teal
- title: "📝 Blog"
  description: "Opinion pieces, build logs, buying advice"
  url: "https://blog.serversatho.me/"
  color: teal
- title: "🛠️ Hire me"
  description: "One-on-one help, migrations, rescue work"
  url: "https://links.serversatho.me/hire"
  color: teal
- title: "GitHub"
  description: "Projects, images, and the source of this wiki"
  url: "https://github.com/serversathome/ServersatHome"
  icon: "img:/_files/github-light.png"
  color: teal
- title: "🐳 Docker Hub"
  description: "Images published by Servers@Home"
  url: "https://hub.docker.com/u/serversathome"
  color: teal
- title: "🧡 Patreon"
  description: "Recurring support and bonus content"
  url: "https://patreon.com/serversathome"
  color: teal
- title: "Buy me a coffee"
  description: "One-time support"
  url: "https://buymeacoffee.com/serversathome"
  icon: "img:/_files/buy-me-a-coffee.png"
  color: teal
```
::

> [!TIP]
> **See something missing or wrong?** Edit the page on [GitHub](https://github.com/serversathome/wiki) and open a pull request, or sign up for an account here to become an editor.
