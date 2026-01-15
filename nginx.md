---
title: Nginx Proxy Manager
description: 
published: true
date: 2026-01-15T15:30:24.739Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:06:44.152Z
---

# ![](/nginx-proxy-manager.png){class="tab-icon"} What is Nginx Proxy Manager?

Nginx Proxy Manager (NPM) is a tool that lets you expose your private web services on your network with free SSL, Docker, and multiple users. You can configure and manage your proxy hosts with a beautiful UI and a simple Docker image.

# 1 · Deploy NPM
# {.tabset}
## <img src="/truenas.png" class="tab-icon"> TrueNAS
![screenshot_from_2025-02-23_07-08-14.png](/screenshot_from_2025-02-23_07-08-14.png)

1. Change the **User ID** and **Group ID** to **0** since Nginx needs to run as root to function.
1. Change the **HTTP Port** to **80** and the **HTTPS Port** to **443**. Choose any port you want for the **WebUI Port** which isn't in use.
1. For the **Nginx Proxy Manager Data Storage** and the **Nginx Proxy Manager Certs Storage** use **Host Path** pointed to a dataset with the correct permissions. 

> The dashboard is now deployed on `http://{serverIP}:81`. The username is *admin@example.com* and the password is *changeme*. After you login you will be forced to change your password.
{.is-info}

## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
 app:
   image: 'jc21/nginx-proxy-manager:latest'
   restart: unless-stopped
   container_name: npm
   ports:
     - '80:80'
     - '81:81'
     - '443:443'
   volumes:
     - /mnt/tank/configs/npm/data:/data
     - /mnt/tank/configs/npm/letsencrypt:/etc/letsencrypt
```

> The dashboard is now deployed on `http://{serverIP}:81`. The username is *admin@example.com* and the password is *changeme*. After you login you will be forced to change your password.
{.is-info}


# 2 · Port Forwarding

In order to reverse proxy your content, you must have ports 80 and 443 properly forwarded to the IP of the host running NPM as well as have a fully qualified domain name (FQDN) DNS entry pointed at the external IP of the router.

# 3 · Domain Name

Whoever manages your domain should have an area for the DNS entries. In order to use nginx properly, we need two records in our DNS table. The first is the A record which points anyone going to example.com to the IP of our server. The second is a CNAME record for \*.example.com which means any subdomain (like cloud.example.com) will also point to the IP address of example.com. The picture below is what this looks like on Cloudflare.

![](/screenshot_from_2024-08-02_09-09-50.png)

# 4 · Add Proxy Hosts
# {.tabset}
## Details

To setup a reverse proxy to a container, add a proxy host. 

![](/screenshot_from_2024-04-15_14-56-45.png)

Add the domain, the internal IP and port number

## Custom Locations
In the event you want to edit the headers, that can only be done in Custom Locations. For example, in the event you want to [bypass caching for media servers](https://wiki.serversatho.me/en/Emby#using-npm).

![screenshot_from_2025-03-28_07-50-43.png](/screenshot_from_2025-03-28_07-50-43.png)

## SSL

Now request a new SSL:

![](/screenshot_from_2024-04-15_14-58-33.png)

You don't have to move these sliders - but it is highly recommended

Done! Now navigate to your site and you are connected via SSL by Let's Encrypt! The certs auto-renew before the 90 days are up so do not worry about updating them manually.

## Advanced

Add this line to increase upload limits: `client_max_body_size 5G;`

![](/screenshot_from_2025-02-06_18-03-27.png)

# <img src="/youtube.png" class="tab-icon"> YouTube Walkthrough

[https://youtu.be/\_FcC79zMiJc](https://youtu.be/mvT0Ehz4s8o)