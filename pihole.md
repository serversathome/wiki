---
title: Pi-Hole
description: A guide to deploying Pi-Hole
published: true
date: '2026-11-04T16:24:08.848Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:07:14.829Z'
---

# What is Pi-Hole?
Pi-hole is a software that blocks ads and trackers across your entire network.

# 1 · Deploy Pi-Hole
:::block-tabs
::block-tab{label="TrueNAS" icon="simple-icons:truenas"}
1. Set the **Storage Configuration** to use **Host Path** for the **Pi-Hole Config Storage** and **Pi-Hole DNSMASQ Configuration**

::

::block-tab{label="Docker Compose" icon="simple-icons:docker"}
```yaml
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80/tcp"
      - "443:443/tcp"
    environment:
      TZ: 'America/New_York'
      FTLCONF_webserver_api_password: 'changeme'
      FTLCONF_dns_listeningMode: 'all'
      PIHOLE_UID: 568
      PIHOLE_GID: 568
    volumes:
      - '/mnt/tank/configs/pihole:/etc/pihole'
    restart: unless-stopped
```

::

:::

# 2 · Logging In
1. Naviagte to `https://{IP}/admin/login`
1. Enter the password you set above

# 3 · Pi-Hole Configuration
1. Navigate to **System → Settings** in the left pane
a. In the **DNS** tab select the DNS servers you want to use

# 4 · Adding Block Lists
1. Navigate to **Group Management → Lists** in the left pane
1. Paste the URL in the **Addresses** field then click **Add Blocklist**
> [!NOTE]
> A basic list to add is EasyList `https://v.firebog.net/hosts/Easylist.txt`

3. Navigate to **System → Tools → Update Gravity** in the left pane
1. Click **Update** and do not navigate away from the page!


# :logos:youtube-icon: 3 · Video





