---
title: Proxmox
description: An overview of the Proxmox hypervisor
published: true
date: 2025-06-09T10:16:16.360Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:26:36.274Z
---

![](/proxmox.png)

![](https://wiki.hydrology.cc/proxmox-ve-8-1-host-summary-secure-boot.png)
# What Is It?

Proxmox Virtual Environment is a complete, open-source server management platform for enterprise virtualization. It tightly integrates the KVM hypervisor and Linux Containers (LXC), software-defined storage and networking functionality, on a single platform. With the integrated web-based user interface you can manage VMs and containers, high availability for clusters, or the integrated disaster recovery tools with ease.

# Why Use It?

I am a big proponent of Ubuntu Server for its simplicity, ease of use, and community support. However, I will admit for beginners, the command line can be intimidating and Webmin is still kind of scary looking. Proxmox gives a lot more functionality to Debian in ways which Webmin does not. It also takes almost no overhead and can do containerization just as well as any other distro out there with the added benefit of virtualization, simple backup, and one-click migration. Many people use this hypervisor in small computers for high-availability clustering. 

# Use Scripts to Make It Easier

There is a [repository](https://community-scripts.github.io/ProxmoxVE/) maintained by the community which hosts one-line commands to help automate many of Proxmox's functions. That isn't to say the Proxmox GUI isn't useful, but there are many steps to creating some things which new users may get lost on. These scripts make life a lot simpler and faster for LXC and VM creation, as well as house keeping tasks for the hypervisor itself.

From Tteck: 

> These scripts empower users to create a Linux container or virtual machine interactively, providing choices for both simple and advanced configurations. The basic setup adheres to default settings, while the advanced setup gives users the ability to customize these defaults.
> 
> Options are displayed to users in a dialog box format. Once the user makes their selections, the script collects and validates their input to generate the final configuration for the container or virtual machine.

## Update on tteck

tteck has released a statement that due to his health the repo will no longer be getting maintenance. Read about it [here](https://servers.hydrology.cc/farewell-to-tteck/).

# Scripts

There are probably over 100 scripts in Tteck's repo. Below are the ones I use most frequently, because after I install Proxmox and do some housekeeping, I deploy the Docker-Alpine LXC container and then use Portainer within that for my containers. You can run separate containers for all of your docker apps if you want; I do this because its easier. 

For reference, my alpine-docker container is running Portainer + 20 docker containers and it uses less than 2 GB of RAM. My entire Proxmox node uses 3 GB of RAM.
# {.tabset}
## Housekeeping

<details>
<summary><strong>Proxmox VE Post Install</strong> (click to expand)</summary>

This script provides options for managing Proxmox VE repositories, including:
- Disabling the Enterprise Repo
- Adding/correcting PVE sources
- Enabling the No-Subscription Repo
- Adding the test Repo
- Disabling the subscription nag
- Updating Proxmox VE
- Rebooting the system

Run the command below in the **Proxmox VE Shell**:
- Source code can be found [here](https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/tools/pve/post-pve-install.sh)

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/tools/pve/post-pve-install.sh)"
```
> It is recommended to answer "yes" (y) to all options presented during the process
{.is-info}

</details>

<details>
<summary><strong>Proxmox VE Kernel Clean</strong> (click to expand)</summary>

Cleaning unused kernel images is beneficial for:
- Reducing the length of the GRUB menu
- Freeing up disk space
- Streamlining the boot process

Run the command below in the **Proxmox VE Shell**:
- Source code can be found [here](https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/tools/pve/kernel-clean.sh)

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/tools/pve/kernel-clean.sh)"
```
</details>

<details>
<summary><strong>Proxmox VE Cron LXC Updater</strong> (click to expand)</summary>

This script will add/remove a crontab schedule that updates all LXCs every Sunday at midnight.
- Source code can be found [here](https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/tools/pve/cron-update-lxcs.sh)

**To exclude specific LXCs from updating:**
1. Edit crontab (`crontab -e`)
2. Add CTIDs as shown in this example (-s 103 111):

```bash
0 0 * * 0 PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/tools/pve/update-lxcs-cron.sh)" -s 103 111 >>/var/log/update-lxcs-cron.log 2>/dev/null
```

Run the command below in the **Proxmox VE Shell** to set up:

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/tools/pve/cron-update-lxcs.sh)"
```
</details>


## Docker LXC

### Options to Install Portainer and/or Docker Compose V2

[Docker](https://www.docker.com/) is an open-source project for automating the deployment of applications as portable, self-sufficient containers.

> If the LXC is created Privileged, the script will automatically set up USB passthrough
{.is-info}


To create a new Proxmox VE Docker LXC, run the command below in the **Proxmox VE Shell**.
```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/docker.sh)"
```

⚡ Default Settings: 2GB RAM - 4GB Storage - 2vCPU ⚡

As an alternative option, you can use Alpine Linux and the Docker package to create a Docker LXC container with faster creation time and minimal system resource usage.   
 

To create a new Proxmox VE Alpine-Docker LXC, run the command below in the **Proxmox VE Shell**.   
```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/alpine-docker.sh)"
```

⚡ Default Settings: 1GB RAM - 2GB Storage - 1vCPU ⚡
> 
> Run Compose V2 by replacing the hyphen (-) with a space, using docker compose, instead of docker-compose
{.is-warning}


**Portainer Interface: (https) IP:9443**

# Connecting With Cloudflare Tunnels
A note for those using a CF tunnel to connect to your WebUI, make these changes to **Additional Application Settings** while editing your tunnel to ensure proper connection:
1. Turn the **No TLS Verify** to `ON`
1. Turn the **Disable Chucked Encoding** to `ON`

> If you have **Botfight Mode** enabled you must create a rule to bypass for your IP
{.is-info}


# Video

To see an example of deploying a docker LXC with a tteck script, go here.

[https://youtu.be/ybDX3onvkis](https://youtu.be/ybDX3onvkis)

Also, watch this beginners guide.

[https://youtu.be/b1BztUYB7VI](https://youtu.be/b1BztUYB7VI)
