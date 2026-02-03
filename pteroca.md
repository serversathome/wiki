---
title: PteroCA
description: A guide to deploying Pteroca
published: true
date: 2026-02-03T16:35:15.715Z
tags: 
editor: markdown
dateCreated: 2026-02-03T16:35:03.541Z
---

# <img src="/pterodactyl.png" class="tab-icon"> What is PteroCA?

A client area and billing panel for Pterodactyl game server hosting. Adds automated billing, credit-based payments, and a customer storefront to your existing Pterodactyl installation.

Live demo: https://demo.pteroca.com (login: `demo@pteroca.com` / `PterocaDemo`)

> 
> PteroCA is a **companion panel** — it requires an existing Pterodactyl Panel installation to function.
{.is-info}

# <img src="/docker.png" class="tab-icon"> 1 · Deploy PteroCA

> 
> PteroCA does not have a pre-built Docker image and must use docker build.
{.is-info}

1. Run the following command in the `stacks` directory of your TrueNAS shell:
   ```bash
   git clone https://github.com/PteroCA-Org/panel.git pteroca
   ```
2. Navigate to your container manager and launch the stack

# 2 · First Setup

1. Access the web UI at `http://your-server:8080/first-configuration`
1. Configure database connection
1. Run migrations when prompted
1. Create your admin account

# 3 · Connect to Pterodactyl

1. In Pterodactyl: **Admin → Application API → Create New**
2. Grant all read/write permissions
3. Copy the API key to PteroCA settings
4. Install the PteroCA plugin in your Pterodactyl container:
   ```bash
   docker exec -it <pterodactyl-container-name> composer require pteroca/pterodactyl-plugin
   ```

> 
> Replace `<pterodactyl-container-name>` with your actual Pterodactyl container name. Run `docker ps` to find it.
{.is-info}

# <img src="/youtube.png" class="tab-icon"> 4 · Video

*No video yet — check back soon!*