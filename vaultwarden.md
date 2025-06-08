---
title: Vaultwarden
description: A guide to installing Vaultwarden in docker via compose
published: true
date: 2025-06-08T18:39:41.807Z
tags: 
editor: markdown
dateCreated: 2024-08-16T22:16:52.287Z
---

![](/v2-b3bfda2319d3c269e7b31e6fe81c9fd7_r-1880582152.png)

# What is Vaultwarden?

Vaultwarden is a free and secure password manager that works on any device and platform. It is the community version of Bitwarden, with features like organizations, attachments, API, and more.

# Docker Compose

```yaml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: unless-stopped
    ports:
      - 8090:80
    environment:
      DOMAIN: "https://vault.example.com"  # Your domain here!
      # ADMIN_TOKEN: changeme
    volumes:
      - /mnt/tank/configs/vaultwarden:/data
```

> Change the domain to the FQDN with https you will be accessing from. Leave the `admin_token` commented out until you finish the install.
{.is-info}


# Logging In

Your Vaultwarden server is now ready on whatever port you specified. However, do not go to http://{localhost}:8090. Even though the page will load you will not be able to create an account. You must come in over https, usually using a reverse proxy like Cloudflare tunnels or Nginx. 

# Admin Controls

After you have created your account, to prevent other users from doing so on your server, you must disable to ability to create new account from the admin panel. 
1. Go back to your compose file an uncomment out the `ADMIN_TOKEN` line and relaunch your container
1. Navigate to https://{yourFQDN}.com/admin and uncheck the box that says *Allow new signups*. When you are done, scroll to the bottom and click **Save**
1. Now comment the `ADMIN_TOKEN` line back out and restart the container. 
> To fully disable to Admin panel, you need to remove the line for the admin token from the config.json file within the container.
{.is-warning}


![](/untitled.jpeg)

# YouTube Walkthrough

[https://youtu.be/cWvWIPMoR1M](https://youtu.be/cWvWIPMoR1M)