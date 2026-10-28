---
title: Webmin
description: A guide to installing Webmin on Ubuntu Server LTS
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:09:27.105Z
---

![webmin.png](/webmin.png)

![](/webmindash.png)

# What is Webmin?

Webmin is a web-based server management control panel for Unix-like systems. Webmin allows the user to configure operating system internals, such as users, disk quotas, services and configuration files, as well as modify and control open-source apps, such as BIND, Apache HTTP Server, PHP, and MySQL.

# Install

Run all of these commands in sequence:

```bash
curl -o webmin-setup-repo.sh https://raw.githubusercontent.com/webmin/webmin/master/webmin-setup-repo.sh
sudo sh webmin-setup-repo.sh
sudo apt-get install webmin --install-recommends -y
```

# Login

The dashboard is available at https://{serverIP}:10000. Webmin uses a self-signed certificate by default, so accept the browser warning the first time. The login credentials are the username and password of the user you set up when installing Ubuntu.

# YouTube Video

Check out an example of this running on an Ubuntu LXC:

[https://youtu.be/48kCm7tbo88](https://youtu.be/48kCm7tbo88)