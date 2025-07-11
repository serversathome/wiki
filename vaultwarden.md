---
title: Vaultwarden
description: A guide to installing Vaultwarden in docker via compose
published: true
date: 2025-07-11T10:39:00.841Z
tags: 
editor: markdown
dateCreated: 2024-08-16T22:16:52.287Z
---

# ![](/vaultwarden.png){class="tab-icon"} What is Vaultwarden?

Vaultwarden is a free and secure password manager that works on any device and platform. It is the community version of Bitwarden, with features like organizations, attachments, API, and more.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Vaultwarden

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


# 2 · Logging In

Your Vaultwarden server is now ready on whatever port you specified. However, do not go to http://{localhost}:8090. Even though the page will load you will not be able to create an account. You must come in over https, usually using a reverse proxy like Cloudflare tunnels or Nginx. 

# 3 · Admin Controls

After you have created your account, to prevent other users from doing so on your server, you must disable to ability to create new account from the admin panel. 
1. Go back to your compose file an uncomment out the `ADMIN_TOKEN` line and relaunch your container
1. Navigate to https://{yourFQDN}.com/admin and uncheck the box that says *Allow new signups*. When you are done, scroll to the bottom and click **Save**
1. Now comment the `ADMIN_TOKEN` line back out and restart the container. 
> To fully disable to Admin panel, you need to remove the line for the admin token from the config.json file within the container.
{.is-warning}


![](/untitled.jpeg)

# <img src="/youtube.png" class="tab-icon"> 4 · Video

[https://youtu.be/cWvWIPMoR1M](https://youtu.be/cWvWIPMoR1M)