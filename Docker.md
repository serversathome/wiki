---
title: Docker
description: A guide to install Docker on Ubuntu Server LTS
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:02:20.673Z
---

# ![](/docker.png){class="tab-icon"} What is Docker?

Docker is a set of platform as a service products that use OS-level virtualization to deliver software in packages called containers. The service has both free and premium tiers. The software that hosts the containers is called Docker Engine. It was first released in 2013 and is developed by Docker, Inc.

# 1 · Installation


```bash
sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc docker-buildx podman-docker containerd runc | cut -f1)
sudo apt update
sudo apt install ca-certificates curl -y
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

Or use the [install-docker.sh](https://github.com/serversathome/ServersatHome/blob/main/install-docker.sh) script, which runs the same steps (the `docker.asc` key and a deb822 `docker.sources` file) and works on Ubuntu or Debian. It also removes an old `docker.list` / `docker.gpg` setup left by earlier versions.

Or execute: 
```bash
wget https://raw.githubusercontent.com/serversathome/ServersatHome/refs/heads/main/install-docker.sh
sudo chmod +x install-docker.sh
sudo bash install-docker.sh
```

> If you use ufw or firewalld, be aware that ports published by Docker containers bypass your firewall rules.
{.is-warning}

# 2 · Official Docker Script 

Docker also provides an official install script:
```bash
curl -fsSL https://get.docker.com -o get-docker.sh && sh get-docker.sh
```

# <img src="/youtube.png" class="tab-icon"> 3 · Video Walkthrough
https://youtu.be/KgpbpyxdRw0