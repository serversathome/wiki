---
title: Gluetun
description: Using the Gluetun container to run other containers with a VPN
published: true
date: 2026-01-15T15:29:21.165Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:52.667Z
---

> There have been widespread reports of issues with this container. The main one being it will work for awhile then go down, or the port will be open then close on its own after 10 minutes or less. As such, I am not recommending this for the time being. Use at your own risk - no support will be answered for issues with this container.
{.is-danger}


# What is Gluetun?

Gluetun Docker provides a universal VPN to all docker containers + non-docker apps. It supports [multiple VPN providers](https://github.com/qdm12/gluetun-wiki/tree/main/setup/providers).

# Docker Compose

```yaml
services:
  gluetun:
    image: qmcgaw/gluetun
    container_name: gluetun
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    environment:
      - VPN_SERVICE_PROVIDER=airvpn
      - VPN_TYPE=wireguard
      - WIREGUARD_PRIVATE_KEY=
      - WIREGUARD_PRESHARED_KEY= #Optional depending on provider/config
      - WIREGUARD_ADDRESSES=
      - SERVER_COUNTRIES= #Optional depending on provider/config
      - FIREWALL_VPN_INPUT_PORTS=6881
    ports:
      - 6881:6881/udp
      - 6881:6881/tcp
    restart: unless-stopped
```

# Ports

In order for this to work with any container, you need a port in this section. 6881 is provided for torrenting, but to add Radarr for example you would need a line which looks like `- 7878:7878`. Within the Radarr container do not define any ports. 

To use port forwarding, add the port you would like to forward in the `FIREWALL_VPN_INPUT_PORTS` line. **Be aware that not all VPN providers allow for port-forwarding; this may affect your seeding speeds or prevent you from properly seeding at all.**

To see an example of this with qBittorrent, see the [qBit page](/qBittorrent).

# Container in the same docker-compose.yml

Add `network_mode: "service:gluetun"` to your second container so that it uses the `gluetun` network stack.

There is no need for `depends_on`.

# External container to Gluetun

Add `--network=container:gluetun` when launching the container, provided Gluetun is already running.

# Container in another docker-compose.yml

Add `network_mode: "container:gluetun"` to your *docker-compose.yml*, provided Gluetun is already running.

# Fixing Common Issues

In Discord many people have said this container fixes the Gluetun issue. I have not tested it, but if you are using Gluetun and you get randomly discconnected every so often, try this method: [qSticky](https://github.com/monstermuffin/qSticky).