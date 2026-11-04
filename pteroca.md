---
title: PteroCA
description: A guide to deploying Pteroca
published: true
date: '2026-11-04T23:32:04.775Z'
tags: []
editor: markdown
dateCreated: '2026-02-03T16:35:03.541Z'
---

# What is PteroCA?

A client area and billing panel for Pterodactyl game server hosting. Adds automated billing, credit-based payments, and a customer storefront to your existing Pterodactyl installation.


> [!NOTE]
> PteroCA is a **companion panel** — it requires an existing Pterodactyl Panel installation to function.

# :logos:docker-icon: 1 · Deploy PteroCA

> [!NOTE]
> PteroCA does not have a pre-built Docker image and must use docker build.

1. Run the following command in the `stacks` directory of your TrueNAS shell:
   ```bash
   git clone https://github.com/PteroCA-Org/panel.git pteroca
   ```
2. Navigate to your container manager and launch the stack

> [!WARNING]
> The root `docker-compose.yml` is the project's development/testing stack: PteroCA is on port **8000**, phpMyAdmin on **8080**, and Mailhog on **8025**. For production the project recommends its automatic installer (`curl -sSL https://pteroca.com/installer.sh | bash`) or `docker-compose.prod.yml`, which uses host port **80**, so remap it on TrueNAS.

# 2 · First Setup

::block-steps
1. Access the web UI at `http://your-server:8000/first-configuration`
1. Configure database connection
1. Run migrations when prompted
1. Create your admin account
::

# 3 · Connect to Pterodactyl

::block-steps
1. In Pterodactyl: **Admin → Application API → Create New**
2. Grant all read/write permissions
3. Copy the API key to PteroCA settings
4. Install the PteroCA addon in your Pterodactyl panel. If you followed the [Pterodactyl guide](/pterodactyl), run this in the Pterodactyl LXC's shell:
   ```bash
   cd /var/www/pterodactyl && composer require pteroca-com/pterodactyl-addon && php artisan optimize:clear
   ```
5. Allow PteroCA in Wings, or server consoles won't work. Edit `/etc/pterodactyl/config.yml` on the Wings host and add your PteroCA domain or IP:
   ```yaml
   allowed_origins:
     - 'pteroca.example.com'
   ```
   Then restart Wings with `sudo systemctl restart wings`.
::

# :logos:youtube-icon: 4 · Video

*No video yet — check back soon!*
