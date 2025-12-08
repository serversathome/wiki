---
title: Nutify
description: A guide to deploying Nutify
published: true
date: 2025-12-08T15:43:33.374Z
tags: 
editor: markdown
dateCreated: 2025-12-08T15:38:24.907Z
---

# <img src="/nutify.png" class="tab-icon"> What is Nutify?
Nutify is a comprehensive monitoring system designed to track the health and performance of your Uninterruptible Power Supply (UPS) devices. It provides real-time insights into critical UPS metrics, allowing you to ensure the continuous operation and protection of your valuable equipment. Nutify collects data, generates detailed reports, and visualizes key parameters through interactive charts, all accessible via a user-friendly web interface.
# <img src="/docker.png" class="tab-icon"> 1 · Deploy Nutify

> Note: I am deploying this on Proxmox which is why this compose looks different than others
{.is-info}


```yaml
services:
  nut:
    image: dartsteven/nutify:amd64-latest
    container_name: Nutify
    privileged: true
    cap_add:
      - SYS_ADMIN
      - SYS_RAWIO
      - MKNOD
    devices:
      - /dev/bus/usb:/dev/bus/usb:rwm
    device_cgroup_rules:
      - 'c 189:* rwm'
    volumes:
      - ./Nutify/logs:/app/nutify/logs
      - ./Nutify/instance:/app/nutify/instance
      - ./Nutify/ssl:/app/ssl
      - ./Nutify/etc/nut:/etc/nut
      - /dev:/dev:rw              # Full /dev access improves hotplug handling
      - /run/udev:/run/udev:ro    # Access to udev events
    environment:

      - SECRET_KEY=test1234567890 # for password encryption and decryption in the database
      - UDEV=1                    # Improve USB detection
    ports:
      - 3493:3493
      - 5050:5050
      - 443:443
    restart: always
    user: root
```