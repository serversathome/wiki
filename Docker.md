---
title: Docker
description: A guide to install Docker on Ubuntu Server LTS
published: true
date: 2026-01-15T15:27:25.739Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:02:20.673Z
---

# ![](/docker.png){class="tab-icon"} What is Docker?

Docker is a set of platform as a service products that use OS-level virtualization to deliver software in packages called containers. The service has both free and premium tiers. The software that hosts the containers is called Docker Engine. It was first released in 2013 and is developed by Docker, Inc.

# 1 · Installation


```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg -y
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo \
 "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
 $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
 sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

Or use this script which is the same as above [/docker-install.sh](/docker-install.sh)

Or execute: 
```bash
wget https://raw.githubusercontent.com/imjustleaving/ServersatHome/refs/heads/main/install-docker.sh
sudo chmod +x install-docker.sh
sudo bash install-docker.sh
```
# 2 · Official Docker Script 

Docker also provides an official install script:
```bash
curl -fsSL https://get.docker.com -o get-docker.sh && sh get-docker.sh
```

# <img src="/youtube.png" class="tab-icon"> 3 · Video Walkthrough
[](https://youtu.be/KgpbpyxdRw0)