---
title: Pangolin
description: A guide to installing Pangolin
published: false
date: 2025-06-13T13:21:37.595Z
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

# Installer Configuration
The installer will prompt you for the following basic information. For example:

1. **Base Domain Name:** Enter your base fully qualified domain name (without any subdomains) Example: example.com
1. **Dashboard Domain Name:** The domain where the application will be hosted. This is used for many things, including generating links. You can run Pangolin on a subdomain or root domain. Example: pangolin.example.com
1. **Let's Encrypt Email:** Provide an email address for SSL certificate registration with Lets Encrypt. This should be an email you have access to.
1. **Tunneling:** You can choose not to install Gerbil for tunneling support - in this config it will just be a normal reverse proxy. See how to [use without tunneling](https://docs.fossorial.io/Pangolin/without-tunneling).

## Admin User Setup

You'll need to configure the admin user. This is the first user in the system. You will log in initially with this user.

1. **Admin Email:** Defaults to `admin@yourdomain.com` but can be customized
1. **Admin Password:** Must meet these requirements:
    - At least 8 characters
    - At least one uppercase letter
    - At least one lowercase letter
    - At least one digit
    - At least one special character

## Security Settings

It will ask you to configure some basic security options. For example:

1. **Signup Without Invite:** Choose whether to disable user registration without invites (defaults to disabled). This removes the "Sign Up" button on the login form and is recommended for private deployments.
1. **Organization Creation:** Decide if users can create their own organizations (defaults to enabled)

## Email Configuration

Decide whether to enable email functionality. This allows Pangolin to send transactional emails like OTP or email verification requests.

If enabled, you'll need to provide:
- SMTP host
- SMTP port (defaults to 587)
- SMTP username
- SMTP password
- No-reply email address. This is the sender email address that Pangolin will email from. Many times this should be the same as the username.
