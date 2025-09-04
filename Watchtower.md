---
title: Watchtower
description: A guide on how to install Watchtower for container updates
published: true
date: 2025-09-04T09:31:19.202Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:43:38.252Z
---

# ![](/watchtower.png){class="tab-icon"} What is Watchtower?
Watchtower is a tool that automates the updating of Docker containers by pulling new images and restarting the containers with the same options used during deployment.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Watchtower

```yaml
services:
  watchtower:
    image: nickfedor/watchtower
    container_name: watchtower
    environment:
      - TZ=America/New_York
      - WATCHTOWER_NOTIFICATIONS_HOSTNAME=
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_INCLUDE_STOPPED=true
      - WATCHTOWER_SCHEDULE=0 0 3 * * *
      # - WATCHTOWER_NOTIFICATION_URL=discord://token@webhookid
      # - WATCHTOWER_NOTIFICATIONS=gotify
      # - WATCHTOWER_NOTIFICATION_GOTIFY_URL=https://
      # - WATCHTOWER_NOTIFICATION_GOTIFY_TOKEN=
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

> I have added some custom environment variables to this compose file. For a full explanation of all possible variables, [see the docs](https://watchtower.nickfedor.com/v1.11.8/configuration/arguments/)
{.is-info}

> Watchtower does not update apps from the TrueNAS catalog. To skip those, add this line to the `environment` section:
> `- WATCHTOWER_DISABLE_CONTAINERS=ix*`
{.is-warning}


# 2 · Run On-Command

If you ever need to update your apps outside of the specified schedule, use this command from the shell of the machine hosting watchtower:

```bash
docker exec -it watchtower /watchtower --run-once
```

# 3 · Notifications

## 3.1 Discord

To use the Discord notification uncomment out the top line `WATCHTOWER_NOTIFICATION_URL`

Your Discord Webhook-URL will look like this:

https://discord.com/api/webhooks/**webhookid**/**token**

The shoutrrr service URL should look like this:

discord://`token`@`webhookid`

## 3.2 Gotify

To use Gotify, uncomment out the bottom 3 lines:
`WATCHTOWER_NOTIFICATIONS`
`WATCHTOWER_NOTIFICATION_GOTIFY_URL`
`WATCHTOWER_NOTIFICATION_GOTIFY_TOKEN`