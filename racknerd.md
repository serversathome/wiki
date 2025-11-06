---
title: Racknerd VPS
description: A guide to deploying a Racknerd VPS
published: true
date: 2025-11-06T10:32:16.088Z
tags: 
editor: markdown
dateCreated: 2025-10-14T13:12:23.435Z
---

# <img src="/racknerd.jpg" class="tab-icon"> What is Racknerd?

RackNerd is a global provider of Infrastructure as a Service (IaaS) solutions, offering dedicated servers, virtual private servers (VPS), and colocation services across multiple datacenter locations. They focus on providing reliable hosting solutions with 24/7 support and competitive pricing.

# 1 • Deploying a VPS

1. Create an account on [Racknerd](https://www.racknerd.com/)
1. Use one of the affilate links to get a discount on a VPS:

    - [1 vCPU | 1GB RAM | 20GB SSD | 2TB bandwidth | $10.96/yr](https://my.racknerd.com/aff.php?aff=15328&pid=917)
    - [2 vCPU | 2GB RAM | 30GB SSD | 3.5TB bandwidth | $17.66/yr](https://my.racknerd.com/aff.php?aff=15328&pid=918)
    - [3 vCPU | 3GB RAM | 55GB SSD | 6TB bandwidth | $26.99/yr](https://my.racknerd.com/aff.php?aff=15328&pid=919)
    - [4 vCPU | 4GB RAM | 80GB SSD | 8TB bandwidth | $39.99/yr](https://my.racknerd.com/aff.php?aff=15328&pid=920)
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

This is a very safe way to secure access to your server. In the event you wanted no ports open you could install a VPN like [tailscale](/tailscale), [netbird](/netbird), twingate, or the like which do not require outbound ports to be open. If you choose to do this, you can remove the `sudo ufw limit 22/tcp` from the above command and no ports will be open. 

## 4.1 Security Updates

By default, there are no updates running on the server, which means security patches are not being applied. To automatically enable updates nightly at 3am, run this command in the shell:

```bash
(crontab -l 2>/dev/null; echo "0 3 * * * apt update && apt upgrade -y && apt autoremove -y") | crontab -
```

# 5 • Running & Accessing Docker
1. To install docker, run the command:
    ```bash
    curl -fsSL https://get.docker.com -o get-docker.sh && sh get-docker.sh
    ```
By default, docker ports exposed on the host bypass ufw, so if you have compose stacks which contain lines like:
```yaml
    ports:
      - 8080:8080
```
you would be able to access port 8080 from the public IP. This bypasses our firewall and makes our VPS insecure. As such, you should not have this block in your compose stacks.

## 5.1 Using a Reverse Proxy

A good way to make docker containers on the VPS accessible is with a reverse proxy like [Cloudflare Tunnels](/CloudflareTunnels) or [Nginx Proxy Manager](/nginx). When using this reverse proxy, you have to refer to the container by its hostname and not its port since the typical ` - ports:` block will be commented out of docker container running on the VPS. 

The entry in the reverse proxy will look like `http://container_name:8080`. This also means the reverse proxy and all containers need to share the same docker network, which they will not by default. You must create a docker network and add all containers to it so their hostnames can be used. 

