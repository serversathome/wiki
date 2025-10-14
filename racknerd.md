---
title: Racknerd VPS
description: A guide to deploying a Racknerd VPS
published: true
date: 2025-10-14T13:52:49.478Z
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

1. SSH into your server
1. Run the command:
    ```bash
    sudo ufw default deny incoming && sudo ufw default allow outbound && sudo ufw limit 22/tcp && sudo ufw enable -y && sudo ufw status verbose
    ```

This is a very safe way to secure access to your server. In the event you wanted no ports open you could install a VPN like tailscale, netbird, twingate, or the like which do not require outbound ports to be open. If you choose to do this, you can remove the `sudo ufw limit 22/tcp` from the above command and no ports will be open. 

# 5 • Running & Accessing Docker
1. To install docker, run the command:
    ```bash
    curl -fsSL https://get.docker.com -o get-docker.sh && sh get-docker.sh
    ```
1. 