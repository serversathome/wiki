---
title: Pangolin
description: A guide to installing Pangolin
published: false
date: 2025-06-13T13:14:49.064Z
tags: 
editor: markdown
dateCreated: 2025-06-13T13:04:34.352Z
---

![pangolin.png](/pangolin.png)

# What is Pangolin?
Pangolin at its core is a self-hosted tunneled reverse proxy with identity and access management, designed to securely expose private resources through encrypted WireGuard tunnels running in user space. Think self hosted Cloudflare tunnels.

# Prerequisites
- A Linux system with root access and a public IP address *(we recommend Ubuntu or Debian based systems)*
- A domain name pointed to your server's IP address
- TCP ports 80, 443, and UDP port 51820 exposed to your Linux instance.

# Choosing a VPS
Pangolin is best run from somewhere outside your network, ideally in the cloud. As such, you need to have a VPS to install Pangolin.

A minimal VPS instance with 1 vCPU, 1GB RAM, and 8GB SSD will perform perfectly well for most use cases. In some cases, you may be able to get away with even less.

The Pangolin docs recommend [this option from Rack Nerd](https://my.racknerd.com/cart.php?a=confproduct&i=0) and honestly it's a great choice, but any VPS will do.

# Installation
Pangolin is installed on bare metal using this command:
```bash
wget -O installer "https://github.com/fosrl/pangolin/releases/download/1.5.1/installer_linux_$(uname -m | sed 's/x86_64/amd64/;s/aarch64/arm64/')" && chmod +x ./installer
```

The downloaded files will be named installer in the current directory.

The installer must be run as root. If you're not already root, switch to the root user or use sudo:
```bash
sudo ./installer
```

