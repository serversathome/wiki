---
title: Nutify
description: A guide to deploying Nutify
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:06:52.884Z
---

# <img src="/nutify.png" class="tab-icon"> What is Nutify?
Nutify is a comprehensive monitoring system designed to track the health and performance of your Uninterruptible Power Supply (UPS) devices. It provides real-time insights into critical UPS metrics, allowing you to ensure the continuous operation and protection of your valuable equipment. Nutify collects data, generates detailed reports, and visualizes key parameters through interactive charts, all accessible via a user-friendly web interface.

<div class="glance">
  <div><span>Port</span><b><code>5050</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Nutify

> Note: I am deploying this on Proxmox which is why this compose looks different than others
{.is-info}


```yaml
services:
  nut:
    image: dartsteven/nutify:latest-amd64 # use the latest-<arch> tag matching your CPU
    container_name: Nutify
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - DAC_OVERRIDE
      - FOWNER
      - KILL
      - SETGID
      - SETUID
    security_opt:
      - no-new-privileges:true
    device_cgroup_rules:
      - 'c 189:* rwm'
    volumes:
      - /mnt/tank/configs/nutify/logs:/app/nutify/logs
      - /mnt/tank/configs/nutify/instance:/app/nutify/instance
      - /mnt/tank/configs/nutify/ssl:/app/ssl
      - /mnt/tank/configs/nutify/etc/nut:/etc/nut
      - /dev/bus/usb:/dev/bus/usb:rw  # USB devices for nut-scanner/NUT
      - /run/udev:/run/udev:ro        # Access to udev events
    environment:
      - SECRET_KEY=<your-secret-key>  # required, generate with: openssl rand -hex 32
      - NUTIFY_WEB_USER=nut
      - NUT_SERVICE_USER=nut
      - UDEV=1                        # Improve USB detection
      - SKIP_PERMCHECK=true
      - SSL_ENABLED=false
    ports:
      - "3493:3493"
      - "5050:5050"
    restart: unless-stopped
```

> The old `amd64-latest` tag stopped updating in November 2025; use the `latest-<arch>` tag for your CPU.
{.is-warning}

Navigate to `http://IP:5050` and follow the setup wizard.
