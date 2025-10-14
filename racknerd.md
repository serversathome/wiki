---
title: Racknerd VPS
description: A guide to deploying a Racknerd VPS
published: true
date: 2025-10-14T13:43:42.663Z
tags: 
editor: markdown
dateCreated: 2025-10-14T13:12:23.435Z
---

# <img src="/racknerd.jpg" class="tab-icon"> What is Racknerd?

RackNerd is a global provider of Infrastructure as a Service (IaaS) solutions, offering dedicated servers, virtual private servers (VPS), and colocation services across multiple datacenter locations. They focus on providing reliable hosting solutions with 24/7 support and competitive pricing.

# 1 • Deploying a VPS

1. Create an account on [Racknerd](https://www.racknerd.com/)
1. Use one of the affilate links to get a discount on a VPS:

    - [1 vCPU | 1GB RAM | 20GB SSD | 2TB bandwidth | $10.96/yr](https://my.racknerd.com/aff.php?aff=15328&pid=912)
    - [2 vCPU | 2GB RAM | 30GB SSD | 3.5TB bandwidth | $17.66/yr](https://my.racknerd.com/aff.php?aff=15328&pid=913)
    {.links-list}

1. Set a **Server Label**
1. Optionally add additional CPU/RAM
1. Choose a **Location** closest to you

# 2 • Logging into the Control Panel
1. Once your VPS is deployed, navigate to [this page](https://my.racknerd.com/clientarea.php?action=services) and click on the row showing your VPS
1. At the bottom of the screen, you will see a link to the [Control Panel](https://nerdvm.racknerd.com/) along with your username. Your password will be emailed to you. If you forgot it navigate to [this page](https://my.racknerd.com/clientarea.php?action=emails) and look for the message titled `KVM VPS Login Information`
1. Naviagte to the [Control Panel](https://nerdvm.racknerd.com/) and login

# 3 • Accessing Your Server
1. Note your IP address from [this page](https://my.racknerd.com/clientarea.php?action=services)
1. Open a terminal and use the command (using your IP address from step 3.1)
    ```bash
    ssh root@{IP}
    ```
1. Your password was emailed to you. If you forgot it navigate to [this page](https://my.racknerd.com/clientarea.php?action=emails) and look for the message titled `KVM VPS Login Information`

# 4 • Securing Your Server

By default, Racknerd has no security on their VPS. We will deploy `ufw` (universal firewall) allowing traffic only on port 22 for ssh with limited login attempts to prevent bruteforce attacks.

1. 