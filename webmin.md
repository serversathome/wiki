---
title: Webmin
description: A guide to installing Webmin on Ubuntu Server LTS
published: true
date: 2026-01-15T15:32:27.435Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:09:27.105Z
---

![webmin.png](/webmin.png)

![](https://wiki.hydrology.cc/webmindash.png)

# What is Webmin?

Webmin is a web-based server management control panel for Unix-like systems. Webmin allows the user to configure operating system internals, such as users, disk quotas, services and configuration files, as well as modify and control open-source apps, such as BIND, Apache HTTP Server, PHP, and MySQL.

# Install

Run all of these commands in sequence:

```bash
curl -o setup-repos.sh https://raw.githubusercontent.com/webmin/webmin/master/setup-repos.sh
sh setup-repos.sh
apt-get install webmin -y
```

# Login

The dashboard is available at http://{serverIP}:10000. The login credentials are the username and password of the user you set up when installing Ubuntu.

# YouTube Video

Check out an example of this running on an Ubuntu LXC:

[https://youtu.be/48kCm7tbo88](https://youtu.be/48kCm7tbo88)